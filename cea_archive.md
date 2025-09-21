# cea_archive GitLab Migration


## Introduction

This document explains how to set up the cea_archive GitLab project so that the important Jenkins tests are available.


## Overview

Please follow all of these steps:

1. [Create cea_archive GitLab Project](#1-create-cea_archive-gitlab-project)
2. [Use G2G to Import cea_archive into GitLab, from Gerrit](#2-use-g2g-to-import-cea_archive-into-gitlab-from-gerrit)
3. [Configure GitLab CI](#3-configure-gitlab-ci)
4. [Explanation of Migrated cea_archive Tests](#4-explanation-of-migrated-cea_archive-tests)
5. [Edits you should make to CI scripts](#5-edits-you-should-make-to-ci-scripts)

<br />

## 1. Create cea_archive GitLab Project

Create a new GitLab project for cea_archive by following the official documentation for creating a blank project:
* https://docs.gitlab.com/user/project/

<br />

## 2. Use G2G to Import cea_archive into GitLab, from Gerrit

Once a blank project has been created, the next step is to import the cea_archive project from Gerrit into GitLab. 

To do this, you will need proper credentials to authenticate with GitLab and enable you to push to the blank cea_archive project. Here we use a Personal Access Token (PAT) as our credential. To create a PAT, follow these instructions:
* https://docs.gitlab.com/user/profile/personal_access_tokens/#create-a-personal-access-token

You also need credentials which enable you to clone the sources from Gerrit.

Once you have your credentials, use the Gerrit-to-GitLab (g2g) tool to migrate the cea_archive data from Gerrit to GitLab.

```
g2g \
 --gerrit-url https://gerrit.ccc.ocre.cea.fr/gerrit \
 --gerrit-login '<GERRIT_USER>:<GERRIT_TOKEN>' \
 --gerrit-project cea_archive \
 --gitlab-url https://gitlab-forge.ccc.ocre.cea.fr \
 --gitlab-login '<PAT_TOKEN_NAME>:<PAT_TOKEN_VALUE>' \
 --gitlab-project <cea_archive_GITLAB_NAMESPACE>/<cea_archive_GITLAB_PROJECT_NAME> \
 --noproxy-gerrit \
 --noproxy-gitlab \
 --timeout 60
```

The value of `--gitlab-project`, for us, was `paratools/cea_archive`. Based on our discussions with you, you will use a namespace other than `paratools` to hold the project. Substitute your chosen namespace for `<cea_archive_GITLAB_NAMESPACE>`.


<br />


## 3. Configure GitLab CI

The final step is to configure GitLab CI so that the tests formerly run in Jenkins are available in GitLab.

We have done this migration for you. To take advantage of this work, you need to do the following:
1. Cherry-pick our work into the new cea_archive project
2. Update GitLab settings for the new cea_archive project

<br />

### Cherry-pick our work into your new project

The Jenkins cea_archive tests have been imported by us into our test project `paratools/cea_archive`.
* https://gitlab-forge.ccc.ocre.cea.fr/paratools/cea_archive

On the `paratools/cea_archive` master branch, there is a single commit that you need to cherry-pick into the master branch of your new cea_archive project.

```
Commit ID = c363428b9c8d31dc92c5a454c5d4e7fa5610670b

$> git diff --name-only HEAD^
gitlab-ci/.gitlab-ci.yml
gitlab-ci/ci.sh
gitlab-ci/utilities.sh
```

Simply cherry-pick this single commit onto the master branch of your new cea_archive project.

<br />

### Update GitLab Settings for the new cea_archive project

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

## 4. Explanation of Migrated cea_archive Tests

The following tests have been migrated from Jenkins to GitLab:
1. CentOS-8 Tests

<br />

<img width="1321" height="432" alt="cea-archive-pipeline" src="https://github.com/user-attachments/assets/654aac92-962e-46d7-96f8-9f7227f7f798" />

<br />

CentOS-8 Test Example Output

<br />

<img width="1413" height="668" alt="cea-archive-test-output" src="https://github.com/user-attachments/assets/c2907396-4ecd-4c32-bac0-cdc9c82a65ca" />

<br />

The scripts for these tests are all stored in the `gitlab-ci` folder at the root of the cea_archive project and are included in commit cherry-picked as part of Step 3.

Check `gitlab-ci/gitlab-ci.yml` for the definition of each tests CI job.

The format of gitlab-ci.yml is well documented here:
* https://docs.gitlab.com/ci/yaml/

<br />
<br />

## 5. Edits you should make to CI scripts

You may need to edit some of the values from `gitlab-ci/.gitlab-ci.yml`.

See #1 and #2 in this screenshot showing the contents of `gitlab-ci/.gitlab-ci.yml`:

<img width="1302" height="664" alt="Screenshot 2025-09-21 at 4 09 53 PM" src="https://github.com/user-attachments/assets/126c3314-73e9-4075-8dcd-138fb6a7c0c0" />

<br />

Excerpt #1 identifies the tag used for the GitLab Runner which will run the cea_archive CI jobs. You should either configure your runner to have the given tag, or else change the value of the tag to match you have configured for your runner.

Excerpt #2 identifies the PCOCC parameters used for the job.
* PCOCC_USER_DATA should point to the same cloud-init script used to run the current Jenkins' cea_archive centos-8 job.
* PCOCC_USER_CONF_DIR should point to whatever PCOCC configuration directory contains a definition of the cea_archive-c8 virtual machine template.

<br />

<br />

<br />

<br />
