# Robinhood GitLab Migration


## Introduction

This document explains how to set up the Robinhood GitLab project so that the important Jenkins tests are available.


## Overview

The following steps should be followed:

1. [Create Robinhood GitLab Project](#1-create-Robinhood-gitlab-project)
2. [Use G2G to Import Robinhood into GitLab, from Gerrit](#2-use-g2g-to-import-Robinhood-into-gitlab-from-gerrit)
3. [Configure GitLab CI](#3-configure-gitlab-ci)
4. [Explanation of Migrated Robinhood Tests](#4-explanation-of-migrated-robinhood-tests)
5. [Edits you should make to CI scripts](#5-edits-you-should-make-to-ci-scripts)

<br />

## 1. Create Robinhood GitLab Project

Create a new GitLab project for Robinhood by following the official documentation for creating a blank project:
* https://docs.gitlab.com/user/project/

<br />

## 2. Use G2G to Import Robinhood into GitLab, from Gerrit

Once a blank project has been created, the next step is to import the Robinhood project from Gerrit into GitLab. 

To do this, you will need proper credentials to authenticate with GitLab and enable you to push to the blank Robinhood project. Here we use a Personal Access Token (PAT) as our credential. To create a PAT, follow these instructions:
* https://docs.gitlab.com/user/profile/personal_access_tokens/#create-a-personal-access-token

You also need credentials which enable you to clone the sources from Gerrit.

Once you have your credentials, use the Gerrit-to-GitLab (g2g) tool to migrate the Robinhood data from Gerrit to GitLab.

```
g2g \
 --gerrit-url https://gerrit.ccc.ocre.cea.fr/gerrit \
 --gerrit-login '<GERRIT_USER>:<GERRIT_TOKEN>' \
 --gerrit-project Robinhood \
 --gitlab-url https://gitlab-forge.ccc.ocre.cea.fr \
 --gitlab-login '<PAT_TOKEN_NAME>:<PAT_TOKEN_VALUE>' \
 --gitlab-project <Robinhood_GITLAB_NAMESPACE>/<Robinhood_GITLAB_PROJECT_NAME> \
 --noproxy-gerrit \
 --noproxy-gitlab \
 --timeout 60
```

The value of `--gitlab-project`, for us, was `paratools/Robinhood`. Based on our discussions with you, you will use a namespace other than `paratools` to hold the project. Substitute your chosen namespace for `<Robinhood_GITLAB_NAMESPACE>`.


<br />


## 3. Configure GitLab CI

The final step is to configure GitLab CI so that the tests formerly run in Jenkins are available in GitLab.

We have done this migration for you. To take advantage of this work, you need to do the following:
1. Cherry-pick our work into the new Robinhood project
2. Update GitLab settings for the new Robinhood project

<br />

### Cherry-pick our work into your new project

The Jenkins Robinhood tests have been imported by us into our test project `paratools/Robinhood`.
* https://gitlab-forge.ccc.ocre.cea.fr/paratools/Robinhood

On the `paratools/Robinhood` master branch, there is a single commit that you need to cherry-pick into the master branch of your new Robinhood project.

```
Commit ID = 409cb9de1abc726c6c62a091f1541f3e60e86d12

$> git diff --name-only HEAD^
gitlab-ci/.gitlab-ci.yml
gitlab-ci/ci-build.sh
gitlab-ci/ci-test.sh
gitlab-ci/utilities.sh
```

Simply cherry-pick this single commit onto the master branch of your new Robinhood project.

<br />

### Update GitLab Settings for the new Robinhood project

Update the following GitLab Settings:
1. Update the CI/CD configuration file setting
2. Update the Merge Request settings

<br />

#### Update the CI/CD configuration file setting

On the left menu, go to Settings -> CI/CD -> General Pipelines. Find the text field for setting the "CI/CD configuration file" and set it to "gitlab-ci/.gitlab-ci.yml"

<img width="1386" height="574" alt="CI:CD configuration file" src="https://github.com/user-attachments/assets/c33832cb-b4ff-4a53-8025-4fab8cc8addb" />

<br />

#### Update the Merge Request settings

On the left menu, go to Settings -> Merge Requests. These are the settings that relate to how merge requests are handled. There are many settings and they should be adjusted to reflect how you want merge requests to be handled.

We recommend specifically focusing on the "Merge request approvals" section where you can specify the approval requirements for merge-requests to be merged.

Additionally, if you want to make sure merge-requests cannot be merged unless the CI tests pass, you should check the box for "Pipelines must succeed" under "Merge checks".

<img width="1386" height="547" alt="Merge Request Pipelines Must Succeed" src="https://github.com/user-attachments/assets/c5ccaa30-3721-49bc-a61b-2b01fd08c36a" />

<br />
<br />

## 4. Explanation of Migrated Robinhood Tests

The following tests have been migrated from Jenkins to GitLab:
1. Test-Lustre2.12-TMPFS
2. Test-Lustre2.15-TMPFS
3. Build-Lustre2.12
4. Build-Lustre2.15
5. Test-Lustre2.12-LUSTRE_HSM
6. Test-Lustre2.15-LUSTRE_HSM

<br />

<img width="1413" height="677" alt="robinhood pipelines 1" src="https://github.com/user-attachments/assets/8299ae21-a6dd-4b9e-888c-0d309f619ed2" />

<br />
<br />

**Test-Lustre2.15-LUSTRE_HSM Example Job Output**

<br />

<img width="1413" height="602" alt="robinhood job output 1" src="https://github.com/user-attachments/assets/db794146-d9fd-4ddf-8ffb-27bd5ceb0c15" />

<br />

The scripts for these tests are all stored in the gitlab-ci folder at the root of the Robinhood project and are included in commit cherry-picked as part of Step 3.

Check `gitlab-ci/gitlab-ci.yml` for the definition of each tests CI job.

The format of gitlab-ci.yml is well documented here:
* https://docs.gitlab.com/ci/yaml/

<br />

## 5. Edits you should make to CI scripts

You may need to edit some of the values from `gitlab-ci/.gitlab-ci.yml`.

### Setting the GitLab Runner Tag

The highlighted section of the screenshot below shows the part of `gitlab-ci/.gitlab-ci.yml` used to specify the tag used for the GitLab Runner which will run the robinhood CI jobs. You should either configure your runner to have the given tag, or else change the value of the tag to match you have configured for your runner.

<br />

<img width="1235" height="294" alt="robinhood runner tag" src="https://github.com/user-attachments/assets/06c54b65-fd38-4c47-aa65-2d0f7bce21ed" />

<br />
<br />

### Setting the PCOCC environment variables

The highlighted sections of the screenshot below show the parts of `gitlab-ci/.gitlab-ci.yml` where PCOCC parameters are set.
* PCOCC_USER_DATA should point to the same cloud-init script used to run the current, equivalent Robinhood job in Jenkins
* PCOCC_USER_CONF_DIR should point to whatever PCOCC configuration directory contains a definition of the VM used in PCOCC_CLUSTER_DEFINITON

<br />

<img width="1316" height="585" alt="robinhood pcocc 1" src="https://github.com/user-attachments/assets/a6afa4cd-3f8a-4500-9ec7-35e5fb97a8a6" />
<img width="1316" height="608" alt="robinhood pcocc 2" src="https://github.com/user-attachments/assets/f285ce5c-8545-4736-9a97-307cef195db0" />
<img width="1316" height="601" alt="robinhood pcocc 3" src="https://github.com/user-attachments/assets/42148228-bde0-4649-9d65-924f4f079d21" />

<br />

<br />

<br />

<br />

<br />

<br />

<br />

<br />

<br />
