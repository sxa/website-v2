---
title: Proposal to remove versionless java provides from RPM packages
date: "2024-06-27T17:00:00+00:00"
author: sxa
description: The current Linux RPM installers in the Adoptium yum/dnf repository provide a generic "versionless" package called "java". This proposal will remove that and only provide the versioned ones to improve distribution compatibility
tags:
  - temurin
  - installer
  - linux
---

## Introduction

Temurin is currently shipped in a yum/dnf repository for use on RPM-based
Linux distributions.  At present this packages satisfies the requirements of
any package which has a dependency on `java`.  While this sounds reasonable
there is the risk of a compatibility issue that also means it can not always
be used as a drop-in replacement for a system JDK on all distributions.

Most RPM distribtuions have a specific version of java that they consider to
be the system version, and this is the one that any java packages in the
distribution's repositories are built against, so there is an assumption
that the package providing a package called "java" will be compatible with
the system JDK.  Here is an example from a Fedora 40 system with the Temurin
repositories set up:

```
[root@d58e0763fa13 /]# dnf whatprovides java
Last metadata expiration check: 0:04:00 ago on Mon Jun 24 16:14:00 2024.
java-21-openjdk-1:21.0.2.0.13-3.fc40.x86_64 : OpenJDK 21 Runtime Environment
Repo        : fedora
Matched from:
Provide    : java = 1:21.0.2.0.13-3.fc40

java-21-openjdk-1:21.0.3.0.9-1.fc40.x86_64 : OpenJDK 21 Runtime Environment
Repo        : updates
Matched from:
Provide    : java = 1:21.0.3.0.9-1.fc40

temurin-11-jdk-11.0.23.0.0.9-1.x86_64 : Eclipse Temurin 11 JDK
Repo        : Adoptium
Matched from:
Provide    : java

temurin-17-jdk-17.0.11.0.0.9-1.x86_64 : Eclipse Temurin 17 JDK
Repo        : Adoptium
Matched from:
Provide    : java

temurin-21-jdk-21.0.3.0.0.9-1.x86_64 : Eclipse Temurin 21 JDK
Repo        : Adoptium
Matched from:
Provide    : java

temurin-22-jdk-22.0.1.0.0.8-1.x86_64 : Eclipse Temurin 22 JDK
Repo        : Adoptium
Matched from:
Provide    : java

temurin-8-jdk-8.0.412.0.0.8-1.x86_64 : Eclipse Temurin 8 JDK
Repo        : Adoptium
Matched from:
Provide    : java
```
whereas if you use a versioned one:
```
[root@d58e0763fa13 /]# dnf whatprovides java-17
Last metadata expiration check: 0:04:33 ago on Mon Jun 24 16:14:00 2024.
java-17-openjdk-1:17.0.10.0.7-3.fc40.x86_64 : OpenJDK 17 Runtime Environment
Repo        : fedora
Matched from:
Provide    : java-17 = 1:17.0.10.0.7-3.fc40

java-17-openjdk-1:17.0.11.0.9-1.fc40.x86_64 : OpenJDK 17 Runtime Environment
Repo        : updates
Matched from:
Provide    : java-17 = 1:17.0.11.0.9-1.fc40

temurin-17-jdk-17.0.11.0.0.9-1.x86_64 : Eclipse Temurin 17 JDK
Repo        : Adoptium
Matched from:
Provide    : java-17

[root@d58e0763fa13 /]# 
```

This proposal will bring us into line with other openjdk distributions which
do not have a "provides: java" line in the RPM configuration, but do have
versioned ones such as `java-17` and `java-17-openjdk` which will continue
to work as-is.  This will ensure that on a distribution with, for example,
java 21 as the system JDK, we will not override that if temurin-17-jdk is
installed, which we currently do.  Note that this is independent of whether
it is installed as an alternative which makes the default `java` executable,
and other tools from the JDK, equal to the one that is installed.

## Pros of this approach

Improved compatibility with other openjdk distributions and Linux
distribution.  Reduced risk of destabilising the system JDK by making it
incompatible with packages expecting `java` if an earlier Temurin package is
installed relative to the distribution declared system JDK.

It will also make it easier for Temurin to act as a drop-in replacement for
non-system JDKs on a variety of distributions.

## Cons of this approach

If any existing Temurin users are relying on it providing a package called
`java` then this will cause a problem.  We would be able to provide
instructions for end users to create a suitable transition package if
required.

## Other considerations

- How could we act as a replacement for the system JDK
- What is the system JDK on various distributions
- What do other openjdk distributions "provide"
- Check compatibility of the alternatives priority levels
