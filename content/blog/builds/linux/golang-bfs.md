---
title: "Building Golang versions >= 1.21.0 from source with BoringCrypto"
date: 2023-11-01T11:39:29+05:30
draft: false
tags: ["build-system", "golang", "fips", "security", "platform", "golden base image"]
---

## Why would anyone build Golang from source?

Golang does provide the [Downloads Page](https://go.dev/dl/), which contains the required installations and pre-compiled binaries on most of the prominent platforms. It's convenient, it's quick, and it gets you started with Go programming in no time. However, there are situations where this straightforward approach might not suffice. 

I could think of two situations in which the consumer would prefer building Golang from source. Firstly, consumers may want to build an application using Golang on a platform for which the pre-compiled binaries are not available. While the official Go downloads page covers a wide array of platforms, there might be specific or less common environments where pre-compiled binaries are not readily accessible. In such cases, building Golang from source becomes the go-to solution, ensuring compatibility with these unique setups.

Secondly, there's the matter of security. Golang is an open-source project, distributed under a [BSD-style license](https://go.dev/LICENSE), making it an attractive target for potential supply chain attacks. If an attacker were to compromise an open-source project, the least conspicuous approach might be to replace the binaries while leaving the source code apparently untouched. However, Golang has taken proactive steps to address this concern. They provide [perfectly reproducible and verified Go toolchains](https://go.dev/blog/rebuild), even offering a [daily report](https://go.dev/rebuild). This report ensures that the binaries generated from the source code matches with the published binaries on the downloads. Nevertheless, some security-conscious consumers may still prefer to take matters into their own hands, choosing to build Golang from authentic source. This approach guarantees that the binaries they generate are free from potential backdoors or any other changes not present in the source code, providing an additional layer of security for their projects.


## Why do I have to build Golang from Source?
My reasons align partially with the second rationale, coupled with a distinct requirement for the application I intend to build using Golang—it must be FIPS compliant or FIPS validated. In this context, my application needs to leverage FIPS-verified crypto libraries to meet the stringent security standards. Unfortunately, the native crypto libraries bundled with Golang don't adhere to FIPS (Federal Information Processing Standards) guidelines and aren't FIPS-friendly. To fulfill this crucial requirement and ensure the highest level of security for my project, building Golang from source and integrating the FIPS-compliant BoringCrypto package became a necessity.

### What is FIPS? How and why should one be FIPS validated?
>FIPS is a set of rules that outline the basic security needs of cryptographic modules used in computer and telecommunication systems. Compliance with these rules is mandatory for non-military, government-run vendors, as well as healthcare and finance businesses that utilize cryptographic modules to protect sensitive data. A cryptographic module, according to entrust.com, is “any combination of hardware, firmware, or software that implements cryptographic functions such as encryption, decryption, digital signatures, authentication techniques and random number generation.”

> The publications and documents associated with FIPS are issued by the National Institute of Standards and Technology (NIST), which is basically a huge federal agency within the US Department of Commerce that provides standards for industries, predominantly other government agencies. Their most recent publication of FIPS is known as FIPS 140-2

> In short, FIPS 140-2 Validated means that a product has been reviewed, tested, and approved by an accredited (NIST approved) testing lab. “A product or implementation does not meet the FIPS 140-1 or FIPS 140-2 applicability requirements by simply implementing an approved security function and acquiring algorithm validation certificates.” That’s right, if you want a product to be 100% approved and validated, it has to undergo the entire process through the Cryptographic Module Validation Program (CMVP) where it comes out pretty and stamped with official validation. 

> If your product is being sold to a US government agency or to an organization that is linked to the government, it must be FIPS 140-2 validated, but FIPS validation/compliance has become extremely common in private sectors as well. The same goes for any product that handles sensitive data in healthcare and finance.

**Ref:** https://www.ipswitch.com/blog/fips-validated-vs-fips-compliant


The pre-packaged native crypto libraries included with Golang do not adhere to FIPS compliance standards, and there are no intentions to alter this status.

> Go’s crypto is not FIPS 140 validated and I’m afraid that there is no possibility of that happening in the future either. I think Ian’s suggestion of using cgo to call out to an existing, certified library is probably your best bet. However, we would not be interested in patches to add hook points all over the Go library, so you would need to carry that work yourself.

**Ref:** https://github.com/golang/go/issues/11658#issuecomment-120441723

The origins of this discussion trace back to the year 2015. Nevertheless, as of the release of Go 1.19 in August 2022, BoringCrypto has been integrated into the main branch of the Go repository at https://github.com/golang/go. Consequently, it is set to be included in all subsequent Golang releases.


### What is BoringSSL and BoringCrypto?

> BoringSSL is a fork of OpenSSL that is designed to meet Google’s needs.

**Ref:** https://kupczynski.info/posts/fips-golang/

**BoringCrypto** is the specific implementation of BoringSSL used within the Go programming language to provide cryptographic capabilities while emphasizing security and compliance with stringent standards.

However, Boringcrypto is not officially supported, and we make no statements about the suitability of it for FIPS 140 compliance. 

> We have been working inside Google on a fork of Go that uses BoringCrypto (the core of BoringSSL) for various crypto primitives, in furtherance of some work related to FIPS 140. We have heard that some external users of Go would be interested in this code as well, so we have published this code here in the main Go repository behind the setting GOEXPERIMENT=boringcrypto.
Use of GOEXPERIMENT=boringcrypto outside Google is unsupported. This mode is not part of the Go 1 compatibility rules, and it may change incompatibly or break in other ways at any time.
To be clear, we are not making any statements or representations about the suitability of this code in relation to the FIPS 140 standard. Interested users will have to evaluate for themselves whether the code is useful for their own purposes.

**Ref:**
- https://go.dev/src/crypto/internal/boring/README
- https://groups.google.com/g/golang-dev/c/fqwZgtzHbzk
- https://boringssl.googlesource.com/boringssl/+/master/crypto/fipsmodule/FIPS.md


## How I built Golang from source?

> The Go toolchain is written in Go. To build it, you need a Go compiler installed. The scripts that do the initial build of the tools look for a "go" command in $PATH, so as long as you have Go installed in your system and configured in your $PATH, you are ready to build Go from source. Or if you prefer you can set $GOROOT_BOOTSTRAP to the root of a Go installation to use to build the new Go toolchain; $GOROOT_BOOTSTRAP/bin/go should be the go command to use.

**Ref:** 
- https://go.dev/doc/install/source
- https://go.dev/doc/toolchain
- https://go.dev/blog/toolchain

The problem with following the procedure outlines in the documentation is that it doesn't work with the latest Golang versions. The documentation is not updated to reflect the changes in the build process. The documentation is still referring to the old build process, which is not compatible with the latest Golang versions and hence I was getting the error:
`
./cmd/dist: found packages build.go (main) and notgo117.go (building_Go_requires_Go_1_17_13_or_later) in /usr/local/go/src/cmd/dist error
`

So, here are the steps I took to resolve the issue and build Golang from sources.

I performed this activity on a Ubuntu 22.04 docker image tag (https://hub.docker.com/layers/library/ubuntu/22.04/images/sha256-ffa841e85005182836d91f7abd24ec081f3910716096955dcc1874b8017b96c9?context=explore). The steps are as follows:
1. Install the pre-requisites for the build: "ca-certificates git netbase build-essential" from the apt package manager.
2. Clone and build the go version 1.4 branch from the official repository. 
3. Clone and build the go version 1.17.13 branch from the official repository using go version 1.4.
4. Clone and built the go version 1.21.x branch from the official repository using go version 1.17.13 and setting the environment variable GOEXPERIMENT=boringssl.

Here is the bash script to do the same:

```
INSTALLS="ca-certificates git netbase build-essential"
apt-get -qqy --no-install-recommends install $INSTALLS

GOLANG_REPOSITORY_URL="https://github.com/golang/go"
BOOTSTRAP1_VERSION="go1.4"
BOOTSTRAP1_PATH="/root/$BOOTSTRAP1_VERSION"
git clone --depth 1 $GOLANG_REPOSITORY_URL -b "release-branch.$BOOTSTRAP1_VERSION" $BOOTSTRAP1_PATH
cd "$BOOTSTRAP1_PATH/src" && ./make.bash

BOOTSTRAP2_VERSION="go1.17"
BOOTSTRAP2_PATH="/root/$BOOTSTRAP2_VERSION"
git clone --depth 1 $GOLANG_REPOSITORY_URL -b "release-branch.$BOOTSTRAP2_VERSION" $BOOTSTRAP2_PATH
cd "$BOOTSTRAP2_PATH/src" && ./make.bash

export GOROOT_BOOTSTRAP=$BOOTSTRAP2_PATH
export GOEXPERIMENT=boringcrypto
git clone --depth 1 $GOLANG_REPOSITORY_URL -b $GOLANG_VERSION $GOROOT
cd $GOROOT/src && ./all.bash

rm -rf $BOOTSTRAP1_PATH $BOOTSTRAP2_PATH
```

PS: *As you can see, I have invoked ./all.bash and not ./make.bash. This is for the reason that ./all.bash will build the golang binaries as well as run the tests. This will ensure that the build is successful and the tests are passing.*

Ref: https://stackoverflow.com/questions/76029504/cannot-build-and-install-go-from-source-code-from-another-tag

#### Stats

Generated Docker Image Size: **3.15 GB**

Time taken by the Build: **18 min 42 sec**

## Verification of the Built Go

1. Validation of the Golang version through `go version` command:
```
go version go1.21.3 X:boringcrypto linux/amd64
```

2. Execution of the following code snippet:
```
package main

import (
    "fmt"
    _ "crypto/tls/fipsonly"
)

func main() {
    fmt.Println("Hello FIPS")
}
```

Works fine for the golang version `go version go1.21.3 X:boringcrypto linux/amd64` built from sources. However, it fails for the pre-compiled binaries from the official downloads page.

**Ref:** https://stackoverflow.com/questions/55466891/verify-fips-mode-in-golang-boringssl

3. Symbols in the generated Golang Build Binary
```
go tool nm go | grep 'fips'
  f2414c D crypto/internal/boring/fipstls.required
  edf000 D crypto/tls.fipsSupportedSignatureAlgorithms
```













