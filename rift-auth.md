# Rift Auth Guide


## Introduction

This document provides instructions for using the new `rift auth` command.


## Contents

1. [Purpose of Rift Auth](#1-purpose-of-rift-auth)
2. [Design Considerations](#2-design-considerations)
3. [Obtaining Rift-Auth](#3-obtaining-rift-auth)
4. [Auth Settings in project.conf](#4-auth-settings-in-projectconf)
5. [Authenticating with Rift Auth](#5-authenticating-with-rift-auth)

<br />


## 1. Purpose of Rift Auth

`rift auth` is an enhancement to Rift that allows users to authenticate themselves and obtain credentials which enable them to access the S3 staging annex.

Support for the S3 staging annex is a new feature of Rift. The feature is available upstream as part of the cea-hpc/rift project on the master branch, since the following PR is now merged:
* [ ParaTools Enhancements to Rift for S3-Based Annex, and more #29 ](https://github.com/cea-hpc/rift/pull/29)

The staging annex is defined in the Ocean project.conf via a new property:
* (required) `staging_annex` - The HTTP(S) URL designating the S3 endpoint, bucket, and prefix to be used for the staging annex.
  * This URL should take the following format: https://<s3-server>/<s3-bucket>/<prefix>

It is important to keep in mind that `rift auth` pathway and its associated credentials are specifically for use with the S3 `staging_annex` as specified in project.conf.

<br />


## 2. Design Considerations

The design of `rift auth` was discussed on the now-closed pull request below. Please see this thread for more information on why certain decisions were made.
* [Enable use of S3-based Annexes #1](https://github.com/eugeneswalker/rift/pull/1)

<br />


## 3. Obtaining Rift Auth

`rift auth` is an enhancement to Rift that is not-yet-merged into upstream Rift.

There are two options for accessing `rift auth`:
- Obtain and install the enhanced Rift from source code using the ParaTools, Inc. fork of Rift
- Obtain and run the pre-built ocean-container Docker image which has the enhanced version of Rift pre-installed

The ParaTools, Inc. fork of Rift can be found here:
* GitHub project: https://github.com/eugeneswalker/rift
* Please see the releases section: https://github.com/eugeneswalker/rift/releases
* Use the latest release tag

The ocean-container Docker image can be accessed by following the instructions in the main README.md here:
* https://github.com/ParaToolsInc/ocean-docs/blob/main/README.md#1-obtain-the-ocean-docker-image
* https://github.com/ParaToolsInc/ocean-docs/blob/main/README.md#2-launch-the-container

<br />


## 4. Auth Settings in Project.conf

`rift auth` makes use of a number of settings defined in the Ocean `project.conf` file:
* (required) `idp_app_token` - The application-specific token used as part of the request to IDP auth endpoint
* (required) `idp_auth_endpoint` - The IDP auth endpoint
* (required) `s3_auth_endpoint` - The endpoint responsible for issuing the final S3 credentials.
* (optional) `s3_credential_file` - Defines the location where obtained credentials are stored. The default value is `~/.rift/auth.json`

Working values of these settings can be found in the `project.conf` committed to the master branch of the paratools/ocean project, hosted on CEA's GitLab-Forge.
* https://gitlab-forge.ccc.ocre.cea.fr/paratools/ocean (access restricted)

<br />


## 5. Authenticating with Rift Auth

The `rift-auth` authentication pathway is automatically invoked whenever another Rift command detects that authentication is necessary. For instance, if a user tries to push sources to the S3 staging annex using `rift annex push`, the `rift auth` pathway will automatically be invoked. Alternatively, a user can explicitly invoke `rift auth` before running other Rift command which require authentication. 

### A. Username/Password Authentication

Once you have an enhanced version of Rift that supports `rift auth` and a `project.conf` with the required authentication settings, you can use `rift auth` to obtain S3 credentials using username and password using the command below.

This output was obtained from a running inside the `ocean-container:latest` Docker image.

First, see that we are running these commands from within the Ocean repository where we have an updated `project.conf`:
```
[root@1f699276056b ocean]# git remote -v
origin	https://<TOKEN>:<VALUE>@gitlab-forge.ccc.ocre.cea.fr/paratools/ocean.git (fetch)
origin	https://<TOKEN>:<VALUE>6@gitlab-forge.ccc.ocre.cea.fr/paratools/ocean.git (push)

[root@1f699276056b ocean]# head -6 project.conf
# Authentication
idp_app_token: ...
idp_auth_endpoint: ...

s3_credential_file: ~/.rift/auth.json
s3_auth_endpoint:  https://annexe-forge.ccc.ocre.cea.fr
```

We can now run `rift auth` from within the Ocean repository directory:
```
[root@1f699276056b ocean]# rift auth
Username [root]: peyralansl
Password:
> succesfully authenticated; token expires in Wed Oct 08 21:25:12 2025
```

When `rift auth` is successfully used with a username and password, a message is displayed letting the user know when the obtained token expires. The obtained credentials are automatically stored at the location set by `s3_credential_file` and will be re-used until they expire. Alternatively, the credential file may be deleted manually in order to force a new credential to be obtained.


### B. Using Pre-Obtained S3 Credentials to Bypass Username/Password Authentication

In some cases, a user may want to use S3 credentials they have obtained on their own, without `rift auth`. It is possible to use externally obtained S3 credentials by setting the following environment variables:

```
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

In the presence of these environment variables, `rift auth` simply uses them insteaad of trying to fetch new credentials or use any non-expired credential found in `s3_credential_file`.

```
[root@1f699276056b ocean]# export AWS_ACCESS_KEY_ID=<MASKED>
[root@1f699276056b ocean]# export AWS_SECRET_ACCESS_KEY=<MASKED>
[root@1f699276056b ocean]# export AWS_SESSION_TOKEN=<MASKED>

[root@1f699276056b ocean]# rift auth
> succesfully authenticated; token expiration time is unknown
```

If `rift auth` is invoked with `-v, verbose` option, there are messages that show what is happening:
```
[root@1f699276056b ocean]# rift -v auth
INFO     found AWS S3 variables in environment; will bypass credentials file
INFO     to allow use of credential file, please clear these environment variables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN
> succesfully authenticated; token expiration time is unknown
```

The token expiration time will not be known in this case of an externally-obtained S3 credential.


