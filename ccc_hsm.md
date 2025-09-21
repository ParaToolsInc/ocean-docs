# ccc_hsm GitLab Migration


## Introduction

This document explains how to set up the ccc_hsm GitLab project so that the important Jenkins tests are available.


## Overview

The following steps should be followed:

1. [Create ccc_hsm GitLab Project](#1-create-ccc_hsm-gitlab-project)
2. [Use G2G to Import ccc_hsm into GitLab, from Gerrit](#2-use-g2g-to-import-ccc_hsm-into-gitlab-from-gerrit)
3. [Configure GitLab CI](#3-configure-gitlab-ci)
4. [Explanation of Migrated ccc_hsm Tests](#4-explanation-of-migrated-ccc_hsm-tests)
5. [Edits you should make to CI scripts](#5-edits-you-should-make-to-ci-scripts)

<br />

## 1. Create ccc_hsm GitLab Project

Create a new GitLab project for ccc_hsm by following the official documentation for creating a blank project:
* https://docs.gitlab.com/user/project/

<br />

## 2. Use G2G to Import ccc_hsm into GitLab, from Gerrit

Once a blank project has been created, the next step is to import the ccc_hsm project from Gerrit into GitLab. 

To do this, you will need proper credentials to authenticate with GitLab and enable you to push to the blank ccc_hsm project. Here we use a Personal Access Token (PAT) as our credential. To create a PAT, follow these instructions:
* https://docs.gitlab.com/user/profile/personal_access_tokens/#create-a-personal-access-token

You also need credentials which enable you to clone the sources from Gerrit.

Once you have your credentials, use the Gerrit-to-GitLab (g2g) tool to migrate the ccc_hsm data from Gerrit to GitLab.

```
g2g \
 --gerrit-url https://gerrit.ccc.ocre.cea.fr/gerrit \
 --gerrit-login '<GERRIT_USER>:<GERRIT_TOKEN>' \
 --gerrit-project ccc_hsm \
 --gitlab-url https://gitlab-forge.ccc.ocre.cea.fr \
 --gitlab-login '<PAT_TOKEN_NAME>:<PAT_TOKEN_VALUE>' \
 --gitlab-project <ccc_hsm_GITLAB_NAMESPACE>/<ccc_hsm_GITLAB_PROJECT_NAME> \
 --noproxy-gerrit \
 --noproxy-gitlab \
 --timeout 60
```

The value of `--gitlab-project`, for us, was `paratools/ccc_hsm`. Based on our discussions with you, you will use a namespace other than `paratools` to hold the project. Substitute your chosen namespace for `<ccc_hsm_GITLAB_NAMESPACE>`.


<br />


## 3. Configure GitLab CI

The final step is to configure GitLab CI so that the tests formerly run in Jenkins are available in GitLab.

We have done this migration for you. To take advantage of this work, you need to do the following:
1. Cherry-pick our work into the new ccc_hsm project
2. Update GitLab settings for the new ccc_hsm project

<br />

### Cherry-pick our work into your new project

The Jenkins ccc_hsm tests have been imported by us into our test project `paratools/ccc_hsm`.
* https://gitlab-forge.ccc.ocre.cea.fr/paratools/ccc_hsm

On the `paratools/ccc_hsm` master branch, there is a single commit that you need to cherry-pick into the master branch of your new ccc_hsm project.

```
Commit ID = d3959ed5930f12fb416eaa29c706c17b832c6c4e

$> git diff --name-only HEAD^
gitlab-ci/.gitlab-ci.yml
gitlab-ci/ci.sh
gitlab-ci/utilities.sh
```

Simply cherry-pick this single commit onto the master branch of your new ccc_hsm project.

<br />

### Update GitLab Settings for the new ccc_hsm project

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

## 4. Explanation of Migrated ccc_hsm Tests

The following tests have been migrated from Jenkins to GitLab:
1. Lustre-2.15.6-Ocean3.8 Test
2. Lustre-Ocean2.8 Test

<br />

<img width="1386" height="470" alt="ccc_hsm pipeline" src="https://github.com/user-attachments/assets/fb49961f-9938-4956-b89c-f7bf2e393271" />

<br />

<br />

**Lustre-2.15.6-Ocean3.8 Test Example Output**

<br />

<img width="1413" height="438" alt="ccc_hsm job output" src="https://github.com/user-attachments/assets/7ad3d45d-ad1b-44b5-a72e-83dd0d742615" />

<br />

<br />

The scripts for these tests are all stored in the `gitlab-ci` folder at the root of the ccc_hsm project and are included in commit cherry-picked as part of Step 3.

Check `gitlab-ci/gitlab-ci.yml` for the definition of each tests CI job.

The format of gitlab-ci.yml is well documented here:
* https://docs.gitlab.com/ci/yaml/

<br />
<br />

## 5. Edits you should make to CI scripts

You may need to edit some of the values from `gitlab-ci/.gitlab-ci.yml`.

### Setting the GitLab Runner Tag

Section #1 in the screenshot below shows the part of `gitlab-ci/.gitlab-ci.yml` used to specify the tag used for the GitLab Runner which will run the ccc_hsm CI jobs. You should either configure your runner to have the given tag, or else change the value of the tag to match you have configured for your runner.

<img width="1302" height="285" alt="Screenshot 2025-09-21 at 4 18 24 PM" src="https://github.com/user-attachments/assets/c6e3d10c-311f-41e2-bac0-d31b5aae1438" />

<br />
<br />

### Setting the PCOCC environment variables

The highlighted sections of the screenshot below show the parts of `gitlab-ci/.gitlab-ci.yml` where PCOCC parameters are set.
* PCOCC_USER_DATA only needs to point to a cloud-init script which installs the CI users SSH key in the VM
* PCOCC_USER_CONF_DIR should point to whatever PCOCC configuration directory contains a definition of the VM used in PCOCC_CLUSTER_DEFINITON

<img width="1413" height="727" alt="ccc_hsm pcocc parameters" src="https://github.com/user-attachments/assets/72f56765-c09f-4382-a20a-dbb546607a08" />

<br />

<br />

<br />

<br />

<br />

<br />
