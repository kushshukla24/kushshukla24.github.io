---
title: "Building Universal Builds for MacOS with Rust"
date: 2023-11-10T09:44:29+05:30
draft: false
tags: ["build-system", "rust", "mac", "universal", "platform"]
---

Updated: 2024-03-07

## What is a Universal Mac Build?

A Universal Mac build refers to a software package or executable that is designed to run on both Intel-based Macs and Apple Silicon (ARM-based) Macs. This concept was introduced by Apple as they transitioned from Intel processors to their custom-designed Apple Silicon processors. It ensures that applications can work efficiently on both architectures without the need for separate versions or emulation.

To create a Universal Mac build

- developers typically use Apple's development tools, like Xcode, and adopt specific practices to ensure their software is compatible with both Intel and Apple Silicon Macs. This often involves compiling code for both architectures and bundling them together in a single executable or package. 

- developers generally need a macOS version that supports Apple Silicon, which is **macOS 11.0 (Big Sur)** or later. macOS Big Sur introduced native support for Apple Silicon, allowing developers to create Universal builds that work on both Intel and Apple Silicon Macs.

  **Ref:** [MacOS 11 | Big Sur | Release Notes](https://developer.apple.com/documentation/macos-release-notes/macos-big-sur-11_0_1-universal-apps-release-notes)

> Native apps run more efficiently than translated apps because the compiler is able to optimize your code for the target architecture. An app that supports only the x86_64 architecture must run under Rosetta translation on Apple silicon. A universal binary runs natively on both Apple silicon and Intel-based Mac computers, because it contains executable code for both architectures.

**Ref:** [Official Documentation | Building a Universal MacOS Binary](https://developer.apple.com/documentation/apple-silicon/building-a-universal-macos-binary)

Creating Universal builds is essential for software developers to provide a seamless experience to users on different Mac hardware. It allows applications to take full advantage of the performance improvements offered by Apple Silicon while maintaining compatibility with older Intel-based Macs.


## Alright, but how to create Universal Mac Build with Rust?
1. Install Rust using the `rustup` tool following the instructions from the official website: https://www.rust-lang.org/tools/install
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- -y
```

2. Install Mac Command Line Tools using the following command:
```bash
xcode-select --install
```

3. Check the installed targets for the active rustup toolchain version using the following command:
```bash 
rustup show
```

4. Make sure that:
```
aarch64-apple-darwin
x86_64-apple-darwin
```
targets are installed. If not, install them using the following commands:
```bash
rustup target add aarch64-apple-darwin
rustup target add x86_64-apple-darwin
```
**PS:** *Rust will already atleast one of the targets installed. So, you might not need to install only one of them.*

5. Build your Rust project twice (one for each target architecture) for `release` using the following command:
```bash
cargo build --release --target x86_64-apple-darwin 
cargo build --release --target aarch64-apple-darwin
```

6. Create a fat Universal binary using the following command:
```bash
lipo -create -output "<output_path>" "target/x86_64-apple-darwin/release/<binary_name>" "target/aarch64-apple-darwin/release/<binary_name>"
```
In case, the target `x86_64-apple-darwin` is default target in the rustup toolchain, the command can be simplified as:
```bash
lipo -create -output "<output_path>" "target/release/<binary_name>" "target/aarch64-apple-darwin/release/<binary_name>"
```

## How to verify the Universal Mac Build?

Since, the Universal Mac Build should support both the architectures, it is important to verify the same. This can be done using the following steps:

Verify the architecture of the generated output using the following command:
```bash
lipo -detailed_info "<output_path>"
```
or
```bash
lipo -archs "<output_path>"
```
or
```bash
file "<output_path>"
```

All the above command should tell that the generated output supports both the architectures.
Something like:
```bash
x86_64 arm64
```

## Size of the Universal Mac Build
Since the Universal Mac Build comprise of both the architectures, it is expected to be larger in size than the individual builds and almost twice of any single architecture. This can be verified using the following command:
```bash
ls -lh "<output_path>"
```

## Consuming the Universal Mac Build
The Universal Mac build created above doesn't work directly when consumed on the other Mac machine. This is because the Universal Mac build contains the paths to the libraries and frameworks that are specific to the machine on which it was built. This can be verified using the following command:
```bash
otool -L "<output_path>"
```
This will show the paths to the libraries and frameworks that are specific to the machine on which it was built.

To consume the Universal Mac build on other Mac machine, the paths to the libraries and frameworks need to be made relative. This can be done using the following set of commands:

1. Extract the architecture specific libraries from the Universal Mac build using the following command:
```bash
lipo -extract aarch64 <universal mac library path> -o <output_path_of_arm64_library>
lipo -extract x86_64 <universal mac library path> -o <output_path_of_amd64_library>
```

2. Change the paths to the libraries and frameworks to be relative using the following command:
```bash
install_name_tool -id <output_path_of_arm64_library> <output_path_of_arm64_library>
install_name_tool -id <output_path_of_amd64_library> <output_path_of_amd64_library>
```

3. Aggregate the architecture specific libraries to create the Universal Mac library using the following command:
```bash
rm -f <universal mac library path> && lipo -create -o <universal mac library path> <output_path_of_arm64_library> <output_path_of_amd64_library>
```

4. Verify the paths to the libraries and frameworks using the following command:
```bash
otool -L "<output_path>"
```

**Ref:** [Understanding dyld @executable_path, @loader_path and @rpath](https://itwenty.me/posts/01-understanding-rpath/)

## Tribulations
1. Using the @rpath, @loader_path, @executable_path, etc. in the Universal Mac Build with the `install_name_tool` command didn't worked for me.

2. The `-change` option of the `install_name_tool` command can be used to change the paths to the libraries and frameworks to be relative. But, it didn't worked for me.




