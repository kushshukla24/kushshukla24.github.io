---
title: "Developer Experience"
date: 2023-08-26T15:39:29+05:30
draft: true
tags: []
---

- Release Engineering
- Build Engineering
- Developer Experience
- Developer Advocate
- DevOps
- Site Reliability Engineering


Developers juggle a lot of things at any given time
Hyperfocus on an issue
Setup should be as productive as possible

ripper: a tool that is used to tear or break something

Cold Release: Deployment/Release with downtime
Hot Release: Deployment/Release without downtime

Cold Release Build: Compiling and building everything from scratch
Hot Release Build: Compiling and building only the changed files

LLVM is a set of compiler and toolchain technologies that can be used to develop a frontend for any programming language and a backend for any instruction set architecture

- cargo invokes rustc once per crate
- rustc decides how many "codegen units" to do in parallel, writes them as .o files, then archives them in an .rlib
- cargo does one final rustc invocation with all the required .rlib, which ends up calling the linker to make a binary

With cargo's --verbose flag, we can see it making separate rustc invocations

In the invocation below, we trace system calls (strace), including children processes (following forks, -f), for the execution of cargo build --quiet. We then redirect standard error (stderr, file descriptor 2) to standard output (stdout, file descriptor 1), and pipe into grep (from g/re/p: Globally search for a Regular Expression and Print matching lines), using "extended" regular expression syntax (-E), and we look for something that starts with execve( and ends with = 0 (which indicates success).
```bash
cargo clean && strace -f -e execve -- cargo build --quiet 2>&1 | grep -E '^execve.*= 0'
```

And if we ask it to use lld, LLVM's linker, instead, it does that!

```bash
cargo clean && RUSTFLAGS="-C link-args=-fuse-ld=lld" strace -f -e execve -- cargo build --quiet 2>&1 | grep -E 'execve\(.*= 0'
```

"Fresh" units would be those that we don't need to recompile.

cargo always tries to build as many crates as possible in parallel, but it can only start the "next" crate if there's enough metadata available about all its dependencies.

For very large Rust projects, and for hot builds, it's not uncommon for linking to make up most of the "build time". And that's why people tend to reach for lld (LLVM's linker).

When building larger applications, linking can become the slowest / most expensive part of a rebuild. GNU ld hasn't been the best option for a while now.

Rui Ueyama changed the game, twice, by working on lld, LLVM's linker, and then on mold, which will be recognized officially by GCC soon!


By default, no debug symbols are included in "release" cargo builds.

Debug symbols allow mapping memory addresses to "source locations", among other things. Which in turns, unlocks comfortable ways of debugging our executable.

we cannot set a breakpoint even if we didn't have debug info.

ELF files (like executable and dynamic libraries, even static libraries) have symbol tables, that are distinct from debug info. The symbol table is what allows us to run nm on an executable and see all the symbols it exports.


symbols are only exported if they need to be exported. In the case of a library, we need to be able to call acos and asin from somewhere else, and so they're in the table. But for an executable, all we need to know (sort of), is where the code starts, and that's the "start address" here.

Debug information is super duper useful, if only so when you get a crash, you can easily map a stack trace (a bunch of memory addresses) to a bunch of source locations (file name / line number, but also function name, etc).

In Cargo.toml, debug = true actually means debug = 2, and it's usually overkill, unless you're doing the sort of debugging where you need to be able to inspect the value of local variables for example. If all you're after is a stack trace, debug = 1 is good enough.

If you think debug information is too large, first off, you're correct, but secondly, try reaching for objcopy --compress-debug-sections to compress it instead of throwing it away with strip: you might find that you're happy with that compromise.

in debug builds, we do very few optimizations, whereas in release builds, we try to optimize as much as possible.

There's another difference: by default, cargo's debug profile enables incremental builds. And cargo's release profile has incremental builds disabled.

with incremental builds, crates are split into more, smaller codegen units.

Incremental builds are super useful to iterate quickly on something. That's why they're enabled by default for debug builds.

If you make a lot of release builds locally, you may want to enable incremental builds for the release profile too.

If the reason you make release builds locally is because you depend on crates that do gzip/bzip2/brotli/zstd decompression for example, and those are super slow in debug, you may want to set up overrides for those dependencies instead!

Unless you explicitly set lto = "off" in your profile in Cargo.toml, cargo performs "thin local LTO", which means it'll try to inline calls across codegen units within the same crate.

But if we want it to be able to inline calls across crates, we have to pick other types of LTO. We can do "fat" LTO, the classical method, and that one should be super expensive

rustc has a built-in profiler (-Z self-profile), and its output can be visualized in a multitude of ways. The measureme repository contains a summarize tool, which shows a table in the CLI, a flamegraph tool which generates a flamegraph, and crox, which converts the profile to Chrome's tracing format.

Those can be viewed in chrome://tracing in a Chromium-based browser, on Perfetto, Speedscope, and more!

## Tools
1. toml2json RustCLI 
2. jq
3. yq
4. tokei: Count lines of code
5. ar: Archiver utility in Unix group of files as a single file.
6. nm (name mangling) is a Unix command used to dump the symbol table and their attributes from a binary executable file
    Similar tool: llvm-nm: lists symbols in different order
7. rustfilt: demangle the Rust symbols
8. ldd: list dynamic dependencies
9. ld: linker
10. strace: trace system calls 
11. rust-gdb
12. objdump: display information from object files
13. crox
14. perfetto



# Follows
1. Rust: https://fasterthanli.me
