---
title: "GCC"
date: 2023-11-11T21:50:29+05:30
tags: ["gcc"]
draft: true
---

Problem:

Phoenix storage services are running on Ubuntu 18.04. The storage services use DRST as a library. This library is a Python library with CPP bindings to AWS CPP SDK for performance reasons. The Python wrapper is written over a specific version of AWS CPP SDK that can be only compiled with GCC version 4.9.3. Any other version leads to successful compilation but encounters issues at runtime. GCC version 4.9.x is available till Ubuntu 18.04 and is not available for further Ubuntu versions.
Approaches:

Install GCC 4.9.3 via available Debian packages on the internet. This led to failures at installation itself due to unresolved and deprecated dependencies. 

Install GCC 4.8 via available Debian packages on the internet. GCC installation was successful but led to a runtime segmentation fault in the libc++.so.6 library that is shipped with the kernel. 

    std::ostream::sentry::sentry(std::ostream&) () from /lib/x86_64-linux-gnu/libstdc++.so.6

    Compile the AWS CPP SDK and DRST libraries statically on Ubuntu 18, so that they can be run on any OS version. This approach was a last resort as this would increase the size of the libraries by a considerable amount and hence increase the container size which is already at ~1.5GB. 

Solution:

Since GCC 4.9.3 is needed for sure, it can be compiled from source code on Ubuntu 22.04 itself.  The catch here is that to build GCC from source, a version of GCC needs to be already installed on the OS. The later versions of GCC can be compiled with the older versions. But since we wanted to install an already outdated version, a certain amount of patching is done to compile GCC 4.9.3 with GCC version 5. GCC 5 is installed from the Xenial repository since it is supported on Ubuntu 18. Once the build is successful, artifact the output and install this every time a build is triggered. Following are the detailed steps of the process:

 

    Create a Ubuntu 22.04 container in interactive mode. 

    Add Xenial repository to the apt sources and install GCC 5:

# Add Xenial to the apt sources
echo 'deb http://archive.ubuntu.com/ubuntu/ xenial main' >> /etc/apt/sources.list
echo 'deb http://archive.ubuntu.com/ubuntu/ xenial universe' >> /etc/apt/sources.list

    apt update && apt install gcc-5 g++-5

Once this is done, we have a GCC version that is greater than the version of GCC we want to compile. 

    Download the source code of GCC 4.9.3, and patch the code base since it needs a few changes to be able to build with GCC 5. Source code is present at https://ftp.gnu.org/gnu/gcc/gcc-4.9.3/gcc-4.9.3.tar.gz

    Patch:

--- libgcc/config/i386/linux-unwind.h 
+++ libgcc/config/i386/linux-unwind.h.patch
@@ -58,7 +58,7 @@
   if (*(unsigned char *)(pc+0) == 0x48
       && *(unsigned long long *)(pc+1) == RT_SIGRETURN_SYSCALL)
     {
-      struct ucontext *uc_ = context->cfa;
+      ucontext_t *uc_ = context->cfa;
       /* The void * cast is necessary to avoid an aliasing warning.
          The aliasing warning is correct, but should not be a problem
          because it does not alias anything.  */

    Once this is done, follow the steps to compile GCC from the source. Note: Based on the number of cores, set an appropriate value for parallel builds in make. Single process build with take almost 4 hours.  

./contrib/download_prerequisites
cd ..
mkdir objdir
cd objdir
../gcc-4.9.3/configure --prefix=$HOME/gcc-4.9.3 --enable-languages=c,c++ --disable-multilib --disable-libsanitizer --disable-libcilkrts
make -j 4 # Setting 4 since this was done on a 8 core machine
make install

This will generate folders like the following which are artifacts of the GCC 4.9.3 build:
root@ubuntu:~/gcc-4.9.3# ls
bin  include  lib  lib64  libexec  share

    Package this as a tarball and keep this as an artifact. Currently, this is kept at http://devhome.druva.org/~phoenixdevbuild/Softwares/dx/gcc-4.9.3-Ubuntu22.04.tar.gz
    The above steps have given us a working GCC 4.9.3 compiled on Ubuntu 22.04. 

    To install this on any build container, follow the steps below:

mkdir -p /root/gcc-4.9/ && \
  cd /root/gcc-4.9 && \
  wget 'http://devhome.druva.org/~phoenixdevbuild/Softwares/dx/gcc-4.9.3-Ubuntu22.04.tar.gz' && \ 
  tar -xvpf gcc-4.9.3-Ubuntu22.04.tar.gz && \
  cd gcc-4.9.3 && \
  cp -r bin/* /usr/bin/ && \
  cp -r include/* /usr/include/ && \
  cp -r lib/* /usr/lib/ && \
  cp -r lib64/* /usr/lib64/ && \
  cp -r libexec/* /usr/libexec/ && \
  cp -r share/* /usr/share && \
  ln -sf /usr/bin/g++ /usr/bin/g++-4.9 && \
  ln -sf /usr/bin/gcc /usr/bin/gcc-4.9 

After these steps, the DRST libraries are built successfully and linked against the required AWS CPP SDK. 