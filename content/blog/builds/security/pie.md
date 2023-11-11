---
title: "Position Independent Code (PIE)"
date: 2023-11-11T21:50:29+05:30
tags: ["security", "pie"]
draft: true
---

Position Independent Code (PIC) describes code that has been built in a way that allows it to run properly regardless of location in memory. This is especially useful for shared libraries, so that they can be loaded in each app memory space. PIE specifically describes executables that have been built entirely this way.

PIE further enables address space layout randomization, thwarting classes of exploits related to memory corruption issues.

In this case, it was discovered that the position-independent executable (PIE) protection is not correctly set on certain binaries which is a good security practice against ROP chain attacks.


Golang

 

build with -buildmode=pie 


## Verification 
check the created binary with checksec utility.

`apt install checksec`

**Requirement**:
Ensure that ASLR, DEP and other Stack protection settings are enabled on Proxy OS distribution as well as all container images that we use (Mostly enabled by default)

Make sure that All binaries are compiled/built with compiler security flags such as ASLR, DEP, Control flow guard etc.

 
## Analysis

How these flags can be enabled during compilation depends on platform.

For e.g. to enable ASLR for golang public documentation suggests we need to add “-buildmode=pie” as compiler option

Control flow guard doesnt seem to be supported yet for golang however it is available for rust, python.