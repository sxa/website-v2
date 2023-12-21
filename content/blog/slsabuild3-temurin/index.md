---
title: SLSA build level 3 compliance on Linux and macOS for Eclipse Temurin
date: "2023-12-19T17:00:00+00:00"
author: sxa
description: Eclipse Temurin by Adoptium is compliant with build level 3 of the SLSA 1.0 secure development framework on Linux and macOS.
tags:
  - temurin
  - security
---

## Introduction

Supply-chain Levels for Software Artifacts, or [SLSA](https://slsa.dev), is a framework with individual levels that software
producers can work towards to make their software more secure, and consumers
can make decisions based on the software package’s security posture.  The
Adoptium project has worked closely with the Eclipse Foundation security
team to work towards making the Eclipse Temurin compliant with the SLSA
specification's build requirements.



[At the end](https://adoptium.net/blog/2022/11/slsa2-temurin/)
[of 2022](https://newsroom.eclipse.org/eclipse-newsletter/2022/december/eclipse-temurin-slsa-level-two-compliant)
we claimed compliance with level 2 of the SLSA v0.1 specification.  Earlier
this year version 1.0 was released and it was split into multiple "tracks",
of which the build track is the only one currently published.  If you're not
familiar with the changes, check out [this lightning
talk](https://youtu.be/uLXzyutZEmQ?si=XjD9H6uO_GEjJVBG) from one of my
colleagues.  We have been able to build on our work done previously to meet
build level 3 for Linux and MacOS for Eclipse Temurin's build and
distribution.

## What have you done since declaring SLSA level 2

We have built on top of the work covered in the earlier blog to meet the
requirements of SLSA build level 3. The additional requirements were as
follows:

### Prevent runs from influencing one another, even within the same project

In order to achieve independence between build runs, we perform all of our
Linux builds in Docker containers. These containers are instantiated, the
build is run and the results saved, and then we shut down the container.
This way there can be no influencing from caching or from one run impacting
a subsequent one.

We have implemented a comparable system on macos by using Orka from
Macstadium which allows us to dynamically spin up virtual machines for each
build run to give us a comparable level of isolation.

For other Operating Systems that we build on - Windows, AIX and Solaris - we
are not currently set up to do something equivalent which is why we are not
claiming SLSA build level 3 for those builds.

### Verifying provenance artifacts

We have introduced a build verification step which can take the SBoM
produced as part of the build output and verify its contents as far as is
practical.  This will do some checks to ensure that the fields are valid and
match expectations about how the product has been built.  This job is stored
in https://github.com/adoptium/temurin-build/blob/master/tooling as
release_download_test.sh which performs SHA and GPG checks as well as
running some basic checks on the downloads.  It also calls
validateSBOMcontent.sh to check the SBoM contents to make sure the
dependencies, including compilers, listed in there match expectations. The
SBoM contents now also includes the SHA256 checksums of all of the build
artifacts.

In addition to all these checks we also verify after each build that the
build code has the features enabled that it should have. This is done using
a custom AQA test job called "smoke tests" which use the tests in the build
repository in the
[buildAndPackage](https://github.com/adoptium/temurin-build/tree/master/test/functional/buildAndPackage)
directory and test various aspects of the built JDK If these checks fail
then these will be trapped early on.

We expect that all of these checks will be enhanced over time, particularly
as we add more details into the SBoM. Note that even now.

### Prevent secret material used to sign the provenance from being accessible to user-defined build steps

The signing jobs that we use are all contained within the jenkins CI system
which we use. These are independent of the build jobs and run as a
subsequent step to avoid the credentials ever being available to the build
jobs.

## What's in the future?

At the moment SLSA build level 3 is the highest level available. We will
look to keep up to date as updates to the specification are made available.
We expect a "level 4" on the build track, and also other tracks to cover
source code.

We are also continuing to work on our reproducible builds which gives an
extra layer of confidence that any customers of Temurin are able to rebuild
from source code in order to independently verify that nothing in our build
systems have been tampered with or introduced any unexpected code.  Anyone
(yes, even you!) can use our fully open-source setup and build scripts to
rebuild the Temurin JDK, and we encourage you to give it a try!

