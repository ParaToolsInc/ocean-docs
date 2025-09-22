# Ocean Contributor Guide


## Introduction

This document provides instructions for contributing to Ocean's new GitLab project.


## Overview

Contributing to the Ocean project can be broken down into these high-level steps.

1. [Obtain the Ocean Docker Image](#1-obtain-the-ocean-docker-image)
2. [Launch a container using the Ocean Docker image](#2-launch-the-container)
3. [Clone the Ocean sources](#3-clone-the-ocean-sources)
4. [Make your source modifications](#4-make-your-modifications)
5. [Test your changes, locally](#5-test-your-changes-locally)
6. [Submit your changes for review via a merge-request](#6-submit-your-merge-request)
7. [Verify that your Merge Request Passes GitLab CI](#7-verify-that-your-merge-request-passes-gitlab-ci)
8. [Update your Merge Request (if needed)](#8-update-your-merge-request-if-needed)
9. [Merge your changes](#9-merge-your-changes)


## Prerequisites

To follow this guide, you will need:
1. Access to Docker
2. Permission to make merge requests to CEA's GitLab Forge Ocean project
3. Permission to push sources to the S3 staging annex (via `rift annex push`)

<br />

## 1. Obtain the Ocean docker image

Fetch the image tarball and load it into Docker using the following commands.

```
$> wget -q https://france.paratools.com/tarballs/ocean-container-latest.tgz

$> docker load --input ocean-container-latest.tgz
Loaded image: ocean-container:latest

$> docker images | grep ocean-container
REPOSITORY                  TAG            IMAGE ID       CREATED         SIZE
ocean-container             latest         81339e29b027   4 days ago      15.4GB
```

<br />

### 2. Launch the container

Once you've obtained the Ocean docker image, here is how you use it to launch a container.

You must use the flags shown below when launching the container in order to take advantage of certain Rift features.

```
$> docker run -it --privileged --device /dev/kvm --name ocean ocean-container:latest

[root@85763b49a628 ~]# rift --version
rift 0.12

[root@85763b49a628 ~]# cat /etc/os-release
NAME="Fedora Linux"
VERSION="39 (Container Image)"
ID=fedora
VERSION_ID=39
VERSION_CODENAME=""
PLATFORM_ID="platform:f39"
PRETTY_NAME="Fedora Linux 39 (Container Image)"
```

<br />

### 3. Clone the Ocean sources

In this example we use a Personal Access Token (PAT) to clone the Ocean sources.

To obtain a PAT, see the instructions here:
* https://docs.gitlab.com/user/profile/personal_access_tokens/#create-a-personal-access-token

```
$> docker run -it --privileged --device /dev/kvm --name ocean ocean-container:latest

[root@85763b49a628 ~]# TOKEN_NAME=...
[root@85763b49a628 ~]# TOKEN_VALUE=...
[root@85763b49a628 ~]# git clone https://$TOKEN_NAME:$TOKEN_VALUE@gitlab-forge.ccc.ocre.cea.fr/paratools/ocean.git
Cloning into 'ocean'...
remote: Enumerating objects: 19848, done.
remote: Counting objects: 100% (34/34), done.
remote: Compressing objects: 100% (34/34), done.
remote: Total 19848 (delta 15), reused 0 (delta 0), pack-reused 19814 (from 1)
Receiving objects: 100% (19848/19848), 41.67 MiB | 34.84 MiB/s, done.
Resolving deltas: 100% (8103/8103), done.

[root@498abeb8f3f3 ~]# cd ocean
[root@498abeb8f3f3 ocean]# git branch
* master
```

<br />

### 4. Make your modifications

We will work towards an example merge-request where the rubygem-text version used in Ocean's master branch is downgraded from v1.3.1 to v1.3.0.

#### Create a new branch

The first step is to create a branch to store our changes.

```
[root@498abeb8f3f3 ocean]# git checkout -b rubygem-text-downgrade-v1.3.0
[root@498abeb8f3f3 ocean]# git branch
  master
* rubygem-text-downgrade-v1.3.0
```

#### Modify the Ocean files

The overall diff will look like this:
```
[root@498abeb8f3f3 ocean]# git diff
diff --git a/packages/rubygem-text/rubygem-text.spec b/packages/rubygem-text/rubygem-text.spec
index c22fa0fe..067341a7 100644
--- a/packages/rubygem-text/rubygem-text.spec
+++ b/packages/rubygem-text/rubygem-text.spec
@@ -1,8 +1,8 @@
-# Generated from text-1.3.1.gem by gem2rpm -*- rpm-spec -*-
+# Generated from text-1.3.0.gem by gem2rpm -*- rpm-spec -*-
 %global gem_name text

 Name: rubygem-%{gem_name}
-Version: 1.3.1
+Version: 1.3.0
 Release: 2%{?dist}
 Summary: A collection of text algorithms
 License: MIT
@@ -67,8 +67,8 @@ popd
 %{gem_instdir}/test

 %changelog
-* Fri May 30 2025 Lyderic Debusschere <lyderic.debusschere@eolen.com> - 1.3.1-2
+* Fri May 30 2025 Lyderic Debusschere <lyderic.debusschere@eolen.com> - 1.3.0-2
 - Add require test

-* Thu Feb 27 2025  <lyderic.debusschere@asplus.fr> - 1.3.1-1
+* Thu Feb 27 2025  <lyderic.debusschere@asplus.fr> - 1.3.0-1
 - Initial package

diff --git a/packages/rubygem-text/sources/text-1.3.0.gem b/packages/rubygem-text/sources/text-1.3.0.gem
new file mode 100644
index 00000000..40268a43
--- /dev/null
+++ b/packages/rubygem-text/sources/text-1.3.0.gem
@@ -0,0 +1 @@
+d5ca93bcf591d60eb4fb3dcd09db1391
\ No newline at end of file

diff --git a/packages/rubygem-text/sources/text-1.3.1.gem b/packages/rubygem-text/sources/text-1.3.1.gem
deleted file mode 100644
index 83b34b43..00000000
--- a/packages/rubygem-text/sources/text-1.3.1.gem
+++ /dev/null
@@ -1 +0,0 @@
-514c3d1db7a955fe793fc0cb149c164f
\ No newline at end of file
```

The following changes are made:
1. packages/rubygem-text/rubygem-text.spec: version changed to v1.3.0, changelog updated
2. packages/rubygem-text/sources/text-1.3.1.gem: deleted
3. packages/rubygem-text/sources/text-1.3.1.gem: added

To properly add the v1.3.0 sources, we have to download the v1.3.0 gemfile and put it in packages/rubygem-text/sources.
```
[root@498abeb8f3f3 ocean]# wget -q https://rubygems.org/downloads/text-1.3.0.gem -P packages/rubygem-text/sources

[root@498abeb8f3f3 ocean]# du -sh packages/rubygem-text/sources/text-1.3.0.gem
136K	packages/rubygem-text/sources/text-1.3.0.gem
```

The sources for v1.3.0 need to be pushed to the Rift staging annex before the commit is made. This way, the actual source contents themselves are not committed to the Ocean repo.

```
[root@498abeb8f3f3 ocean]# rift annex push packages/rubygem-text/sources/text-1.3.0.gem
Username [root]: YOUR-USERNAME
Password:
> packages/rubygem-text/sources/text-1.3.0.gem: moved and replaced
```

Now that the v1.3.0 sources are uploaded, the contents of the gemfile contains onlythe hash of the sources.
```
[root@498abeb8f3f3 ocean]# echo $(cat packages/rubygem-text/sources/text-1.3.0.gem)
d5ca93bcf591d60eb4fb3dcd09db1391
```

<br />

### 5. Test your changes, locally

In order for your changes to ultimately be accepted to Ocean, the GitLab CI for Ocean requires they pass both "RPM Lint" and "Validate Diff" jobs.

You can ensure your changes pass these tests by running them locally, prior to submitting your merge request.

#### Test 1: RPM Linter

We can run the RPM Linter on our changes just as it will be run as part of the automated GitLab CI like this:

```
[root@498abeb8f3f3 ocean]# git --no-pager diff | rift gitlab -
0 packages and 1 specfiles checked; 0 errors, 0 warnings.

[root@498abeb8f3f3 ocean]# echo $?
0
```

#### Test 2: Validate Diff

If the RPM Lint test passes, you can run the "Validate Diff" tests locally just as it will be run as part of automated GitLab CI like this:

```
[root@498abeb8f3f3 ocean]# git --no-pager diff | rift validdiff -
** Checking package 'rubygem-text' on architecture x86_64 **
> Validate package info...
> Validate specfile...
0 packages and 1 specfiles checked; 0 errors, 0 warnings.
> Preparing Mock environment...
> Validate source RPM build...
> Validate RPMS build...
> Preparing x86_64 test environment
> Launching VM ...
Metadata cache created.
No repository match: working

Upgraded:
  e2fsprogs-1.47.2-wc2.ocean1.el9.x86_64
  e2fsprogs-libs-1.47.2-wc2.ocean1.el9.x86_64
  libcom_err-1.47.2-wc2.ocean1.el9.x86_64
  libss-1.47.2-wc2.ocean1.el9.x86_64

** Starting tests of package rubygem-text on architecture x86_64 **
> Running test 'rubygem-text.basic_install' on architecture 'x86_64'
[Testing 'rubygem-text' (1/2)]
Last metadata expiration check: 0:00:36 ago on Sun Sep 21 02:53:29 2025.
Dependencies resolved.
================================================================================
 Package                 Arch        Version                 Repository    Size
================================================================================
Installing:
 rubygem-text            noarch      1.3.0-2.el9             staging       17 k
Installing dependencies:
 ruby                    x86_64      3.0.7-162.el9_4         updates       38 k
 ruby-libs               x86_64      3.0.7-162.el9_4         updates      3.2 M
 rubygem-json            x86_64      2.5.1-162.el9_4         updates       51 k
 rubygem-psych           x86_64      3.3.2-162.el9_4         updates       48 k
 rubygems                noarch      3.2.33-162.el9_4        updates      253 k
Installing weak dependencies:
 ruby-default-gems       noarch      3.0.7-162.el9_4         updates       29 k
 rubygem-bigdecimal      x86_64      3.0.0-162.el9_4         updates       51 k
 rubygem-bundler         noarch      2.2.33-162.el9_4        updates      369 k
 rubygem-io-console      x86_64      0.5.7-162.el9_4         updates       22 k
 rubygem-rdoc            noarch      6.3.4.1-162.el9_4       updates      398 k

Transaction Summary
================================================================================
Install  11 Packages

Total size: 4.4 M
Total download size: 4.4 M
Installed size: 16 M
Downloading Packages:
--------------------------------------------------------------------------------
Total                                           8.1 MB/s | 4.4 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction

Installed:
  ruby-3.0.7-162.el9_4.x86_64
  ruby-default-gems-3.0.7-162.el9_4.noarch
  ruby-libs-3.0.7-162.el9_4.x86_64
  rubygem-bigdecimal-3.0.0-162.el9_4.x86_64
  rubygem-bundler-2.2.33-162.el9_4.noarch
  rubygem-io-console-0.5.7-162.el9_4.x86_64
  rubygem-json-2.5.1-162.el9_4.x86_64
  rubygem-psych-3.3.2-162.el9_4.x86_64
  rubygem-rdoc-6.3.4.1-162.el9_4.noarch
  rubygem-text-1.3.0-2.el9.noarch
  rubygems-3.2.33-162.el9_4.noarch

Complete!
> Cleanup last transaction
Last metadata expiration check: 0:00:40 ago on Sun Sep 21 02:53:29 2025.
Dependencies resolved.
================================================================================
 Package                 Arch        Version                Repository     Size
================================================================================
Removing:
 ruby-default-gems       noarch      3.0.7-162.el9_4        @updates       91 k
 rubygem-bigdecimal      x86_64      3.0.0-162.el9_4        @updates      106 k
 rubygem-bundler         noarch      2.2.33-162.el9_4       @updates      1.3 M
 rubygem-io-console      x86_64      0.5.7-162.el9_4        @updates       33 k
 rubygem-rdoc            noarch      6.3.4.1-162.el9_4      @updates      1.7 M
 rubygem-text            noarch      1.3.0-2.el9            @staging       29 k
Removing dependent packages:
 ruby                    x86_64      3.0.7-162.el9_4        @updates       90 k
 ruby-libs               x86_64      3.0.7-162.el9_4        @updates       12 M
 rubygem-json            x86_64      2.5.1-162.el9_4        @updates      134 k
 rubygem-psych           x86_64      3.3.2-162.el9_4        @updates      135 k
 rubygems                noarch      3.2.33-162.el9_4       @updates      970 k

Transaction Summary
================================================================================
Remove  11 Packages

Freed space: 16 M
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction

Removed:
  ruby-3.0.7-162.el9_4.x86_64
  ruby-default-gems-3.0.7-162.el9_4.noarch
  ruby-libs-3.0.7-162.el9_4.x86_64
  rubygem-bigdecimal-3.0.0-162.el9_4.x86_64
  rubygem-bundler-2.2.33-162.el9_4.noarch
  rubygem-io-console-0.5.7-162.el9_4.x86_64
  rubygem-json-2.5.1-162.el9_4.x86_64
  rubygem-psych-3.3.2-162.el9_4.x86_64
  rubygem-rdoc-6.3.4.1-162.el9_4.noarch
  rubygem-text-1.3.0-2.el9.noarch
  rubygems-3.2.33-162.el9_4.noarch

Complete!
[Testing 'rubygem-text-doc' (2/2)]
Last metadata expiration check: 0:00:41 ago on Sun Sep 21 02:53:29 2025.
Dependencies resolved.
================================================================================
 Package                 Arch        Version                 Repository    Size
================================================================================
Installing:
 rubygem-text-doc        noarch      1.3.0-2.el9             staging      309 k
Installing dependencies:
 ruby                    x86_64      3.0.7-162.el9_4         updates       38 k
 ruby-libs               x86_64      3.0.7-162.el9_4         updates      3.2 M
 rubygem-json            x86_64      2.5.1-162.el9_4         updates       51 k
 rubygem-psych           x86_64      3.3.2-162.el9_4         updates       48 k
 rubygem-text            noarch      1.3.0-2.el9             staging       17 k
 rubygems                noarch      3.2.33-162.el9_4        updates      253 k
Installing weak dependencies:
 ruby-default-gems       noarch      3.0.7-162.el9_4         updates       29 k
 rubygem-bigdecimal      x86_64      3.0.0-162.el9_4         updates       51 k
 rubygem-bundler         noarch      2.2.33-162.el9_4        updates      369 k
 rubygem-io-console      x86_64      0.5.7-162.el9_4         updates       22 k
 rubygem-rdoc            noarch      6.3.4.1-162.el9_4       updates      398 k

Transaction Summary
================================================================================
Install  12 Packages

Total size: 4.7 M
Total download size: 4.4 M
Installed size: 17 M
Downloading Packages:
--------------------------------------------------------------------------------
Total                                            10 MB/s | 4.4 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction

Installed:
  ruby-3.0.7-162.el9_4.x86_64
  ruby-default-gems-3.0.7-162.el9_4.noarch
  ruby-libs-3.0.7-162.el9_4.x86_64
  rubygem-bigdecimal-3.0.0-162.el9_4.x86_64
  rubygem-bundler-2.2.33-162.el9_4.noarch
  rubygem-io-console-0.5.7-162.el9_4.x86_64
  rubygem-json-2.5.1-162.el9_4.x86_64
  rubygem-psych-3.3.2-162.el9_4.x86_64
  rubygem-rdoc-6.3.4.1-162.el9_4.noarch
  rubygem-text-1.3.0-2.el9.noarch
  rubygem-text-doc-1.3.0-2.el9.noarch
  rubygems-3.2.33-162.el9_4.noarch

Complete!
> Cleanup last transaction
Last metadata expiration check: 0:00:45 ago on Sun Sep 21 02:53:29 2025.
Dependencies resolved.
================================================================================
 Package                 Arch        Version                Repository     Size
================================================================================
Removing:
 ruby-default-gems       noarch      3.0.7-162.el9_4        @updates       91 k
 rubygem-bigdecimal      x86_64      3.0.0-162.el9_4        @updates      106 k
 rubygem-bundler         noarch      2.2.33-162.el9_4       @updates      1.3 M
 rubygem-io-console      x86_64      0.5.7-162.el9_4        @updates       33 k
 rubygem-rdoc            noarch      6.3.4.1-162.el9_4      @updates      1.7 M
 rubygem-text-doc        noarch      1.3.0-2.el9            @staging      1.1 M
Removing dependent packages:
 ruby                    x86_64      3.0.7-162.el9_4        @updates       90 k
 ruby-libs               x86_64      3.0.7-162.el9_4        @updates       12 M
 rubygem-json            x86_64      2.5.1-162.el9_4        @updates      134 k
 rubygem-psych           x86_64      3.3.2-162.el9_4        @updates      135 k
 rubygem-text            noarch      1.3.0-2.el9            @staging       29 k
 rubygems                noarch      3.2.33-162.el9_4       @updates      970 k

Transaction Summary
================================================================================
Remove  12 Packages

Freed space: 17 M
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction

Removed:
  ruby-3.0.7-162.el9_4.x86_64
  ruby-default-gems-3.0.7-162.el9_4.noarch
  ruby-libs-3.0.7-162.el9_4.x86_64
  rubygem-bigdecimal-3.0.0-162.el9_4.x86_64
  rubygem-bundler-2.2.33-162.el9_4.noarch
  rubygem-io-console-0.5.7-162.el9_4.x86_64
  rubygem-json-2.5.1-162.el9_4.x86_64
  rubygem-psych-3.3.2-162.el9_4.x86_64
  rubygem-rdoc-6.3.4.1-162.el9_4.noarch
  rubygem-text-1.3.0-2.el9.noarch
  rubygem-text-doc-1.3.0-2.el9.noarch
  rubygems-3.2.33-162.el9_4.noarch

Complete!
> Test 'rubygem-text.basic_install' on architecture x86_64: OK
> Running test 'rubygem-text.ruby_require' on architecture 'x86_64'
Last metadata expiration check: 0:00:46 ago on Sun Sep 21 02:53:29 2025.
Dependencies resolved.
================================================================================
 Package                 Arch        Version                 Repository    Size
================================================================================
Installing:
 rubygem-text            noarch      1.3.0-2.el9             staging       17 k
Installing dependencies:
 ruby                    x86_64      3.0.7-162.el9_4         updates       38 k
 ruby-libs               x86_64      3.0.7-162.el9_4         updates      3.2 M
 rubygem-json            x86_64      2.5.1-162.el9_4         updates       51 k
 rubygem-psych           x86_64      3.3.2-162.el9_4         updates       48 k
 rubygems                noarch      3.2.33-162.el9_4        updates      253 k
Installing weak dependencies:
 ruby-default-gems       noarch      3.0.7-162.el9_4         updates       29 k
 rubygem-bigdecimal      x86_64      3.0.0-162.el9_4         updates       51 k
 rubygem-bundler         noarch      2.2.33-162.el9_4        updates      369 k
 rubygem-io-console      x86_64      0.5.7-162.el9_4         updates       22 k
 rubygem-rdoc            noarch      6.3.4.1-162.el9_4       updates      398 k

Transaction Summary
================================================================================
Install  11 Packages

Total size: 4.4 M
Total download size: 4.4 M
Installed size: 16 M
Downloading Packages:
(1/10): ruby-default-gems-3.0.7-162.el9_4.noarc 126 kB/s |  29 kB     00:00
(2/10): ruby-3.0.7-162.el9_4.x86_64.rpm         143 kB/s |  38 kB     00:00
(3/10): rubygem-bigdecimal-3.0.0-162.el9_4.x86_ 1.1 MB/s |  51 kB     00:00
(4/10): rubygem-bundler-2.2.33-162.el9_4.noarch 5.7 MB/s | 369 kB     00:00
(5/10): rubygem-io-console-0.5.7-162.el9_4.x86_ 413 kB/s |  22 kB     00:00
(6/10): rubygem-json-2.5.1-162.el9_4.x86_64.rpm 1.3 MB/s |  51 kB     00:00
(7/10): rubygem-psych-3.3.2-162.el9_4.x86_64.rp 1.2 MB/s |  48 kB     00:00
(8/10): rubygem-rdoc-6.3.4.1-162.el9_4.noarch.r 7.7 MB/s | 398 kB     00:00
(9/10): rubygems-3.2.33-162.el9_4.noarch.rpm    4.7 MB/s | 253 kB     00:00
(10/10): ruby-libs-3.0.7-162.el9_4.x86_64.rpm   6.6 MB/s | 3.2 MB     00:00
--------------------------------------------------------------------------------
Total                                           9.0 MB/s | 4.4 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : ruby-libs-3.0.7-162.el9_4.x86_64                      1/11
  Installing       : rubygem-bigdecimal-3.0.0-162.el9_4.x86_64             2/11
  Installing       : ruby-default-gems-3.0.7-162.el9_4.noarch              3/11
  Installing       : rubygem-bundler-2.2.33-162.el9_4.noarch               4/11
  Installing       : rubygem-io-console-0.5.7-162.el9_4.x86_64             5/11
  Installing       : rubygem-json-2.5.1-162.el9_4.x86_64                   6/11
  Installing       : rubygem-psych-3.3.2-162.el9_4.x86_64                  7/11
  Installing       : rubygem-rdoc-6.3.4.1-162.el9_4.noarch                 8/11
  Installing       : rubygems-3.2.33-162.el9_4.noarch                      9/11
  Installing       : ruby-3.0.7-162.el9_4.x86_64                          10/11
  Installing       : rubygem-text-1.3.0-2.el9.noarch                      11/11
  Running scriptlet: rubygem-text-1.3.0-2.el9.noarch                      11/11
  Verifying        : rubygem-text-1.3.0-2.el9.noarch                       1/11
  Verifying        : ruby-3.0.7-162.el9_4.x86_64                           2/11
  Verifying        : ruby-default-gems-3.0.7-162.el9_4.noarch              3/11
  Verifying        : ruby-libs-3.0.7-162.el9_4.x86_64                      4/11
  Verifying        : rubygem-bigdecimal-3.0.0-162.el9_4.x86_64             5/11
  Verifying        : rubygem-bundler-2.2.33-162.el9_4.noarch               6/11
  Verifying        : rubygem-io-console-0.5.7-162.el9_4.x86_64             7/11
  Verifying        : rubygem-json-2.5.1-162.el9_4.x86_64                   8/11
  Verifying        : rubygem-psych-3.3.2-162.el9_4.x86_64                  9/11
  Verifying        : rubygem-rdoc-6.3.4.1-162.el9_4.noarch                10/11
  Verifying        : rubygems-3.2.33-162.el9_4.noarch                     11/11

Installed:
  ruby-3.0.7-162.el9_4.x86_64
  ruby-default-gems-3.0.7-162.el9_4.noarch
  ruby-libs-3.0.7-162.el9_4.x86_64
  rubygem-bigdecimal-3.0.0-162.el9_4.x86_64
  rubygem-bundler-2.2.33-162.el9_4.noarch
  rubygem-io-console-0.5.7-162.el9_4.x86_64
  rubygem-json-2.5.1-162.el9_4.x86_64
  rubygem-psych-3.3.2-162.el9_4.x86_64
  rubygem-rdoc-6.3.4.1-162.el9_4.noarch
  rubygem-text-1.3.0-2.el9.noarch
  rubygems-3.2.33-162.el9_4.noarch

Complete!
text loaded successfully
> Test 'rubygem-text.ruby_require' on architecture x86_64: OK
> Cleaning x86_64 test environment
** All packages checked on architecture x86_64 **
NAME                       ARCH   DURATION RESULT
----                       ----   -------- ------
rubygem-text.build         x86_64      16s Success
rubygem-text.basic_install x86_64      11s Success
rubygem-text.ruby_require  x86_64       3s Success
** Test suite SUCCEEDED **
```

<br />

### 6. Submit your Merge Request

Once you have verified your changes pass the tests, you should create a merge request to submit them to Ocean for review.

#### Commit your changes
```
[root@498abeb8f3f3 ocean]# git add packages/rubygem-text

[root@498abeb8f3f3 ocean]# git status -s
M  packages/rubygem-text/rubygem-text.spec
A  packages/rubygem-text/sources/text-1.3.0.gem
D  packages/rubygem-text/sources/text-1.3.1.gem

[root@498abeb8f3f3 ocean]# git config --global user.email your-email
[root@498abeb8f3f3 ocean]# git config --global user.name your-name

[root@498abeb8f3f3 ocean]# git commit -m 'rubygem-text: downgrade to v1.3.0'
[rubygem-text-downgrade-v1.3.0 c9ec7b97] rubygem-text: downgrade to v1.3.0
 3 files changed, 5 insertions(+), 5 deletions(-)
 create mode 100644 packages/rubygem-text/sources/text-1.3.0.gem
 delete mode 100644 packages/rubygem-text/sources/text-1.3.1.gem
```

#### Push your branch to Ocean
```
[root@498abeb8f3f3 ocean]# git push -u origin rubygem-text-downgrade-v1.3.0
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 8 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7), 658 bytes | 658.00 KiB/s, done.
Total 7 (delta 3), reused 0 (delta 0), pack-reused 0 (from 0)
remote:
remote: To create a merge request for rubygem-text-downgrade-v1.3.0, visit:
remote:   https://gitlab-forge.ccc.ocre.cea.fr/paratools/ocean/-/merge_requests/new?merge_request%5Bsource_branch%5D=rubygem-text-downgrade-v1.3.0
remote:
To https://gitlab-forge.ccc.ocre.cea.fr/paratools/ocean.git
 * [new branch]        rubygem-text-downgrade-v1.3.0 -> rubygem-text-downgrade-v1.3.0
branch 'rubygem-text-downgrade-v1.3.0' set up to track 'origin/rubygem-text-downgrade-v1.3.0'.
```

Note that the output above contains the following instructions:
```
remote: To create a merge request for rubygem-text-downgrade-v1.3.0, visit:
remote:   https://gitlab-forge.ccc.ocre.cea.fr/paratools/ocean/-/merge_requests/new?merge_request%5Bsource_branch%5D=rubygem-text-downgrade-v1.3.0
```

Navigate to this URL in your web browser to submit the pull request.

When submitting the pull request, be careful to select the correct target branch which you are requesting your changes be merged in to. For this example, we are targetting our changes to the default branch, master branch. If you need to target a different branch, click "Change branches". See the screenshot below.

<img width="1387" height="576" alt="merge request form" src="https://github.com/user-attachments/assets/2a3021b4-2c2a-4abf-81e5-630df0b6381b" />

<br />

### 7. Verify that your Merge Request Passes GitLab CI

Once submitted, GitLab will automatically test your merge-request as part of CI.

Initially, the merge request will look something like this:

<img width="1387" height="651" alt="Merge-request rubygem-text downgrade" src="https://github.com/user-attachments/assets/13073d48-dc66-4af7-92f1-dabb8ba7c781" />

Note where it says "Merge request pipeline #651 running."

Click the pipeline number to view your merge-request's CI pipeline.

<img width="1387" height="469" alt="Merge-request rubygem-text downgrade pipeline" src="https://github.com/user-attachments/assets/df1b9aab-1a43-47af-8e12-89dea80fe3d1" />

In the screenshot here we see that the RPM Lint job has passed and that the Validate Diff job is running.

Click on a job to view its logs.

Once the Validate Diff job completes, its output should match what was seen when the test was run locally prior to submitting the merge-request:

<img width="1387" height="740" alt="Merge-request rubygem-text downgrade validate-diff job" src="https://github.com/user-attachments/assets/6249274c-b6a0-4eff-86e3-a1eb4957809e" />

<br />
<br />

### 8. Update your Merge Request (if needed)

Any additional commits pushed to your merge-request branch will cause CI to re-run. 

<br />

### 9. Merge your changes

You can merge your changes to the target branch once CI passes and the appropriate number of reviewers have approved it.

<img width="1387" height="627" alt="Merge-request rubygem-text merge" src="https://github.com/user-attachments/assets/2dee6190-bf2a-4b4b-8241-d1f0740bdede" />

<br />

<br />

<br />

<br />
