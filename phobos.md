# Phobos GitLab Migration


## Introduction

This document explains how to set up the Phobos GitLab project so that the important Jenkins tests are available.


## Overview

The following steps should be followed:

1. [Create Phobos GitLab Project](#1-create-phobos-gitlab-project)
2. [Use G2G to Import Phobos into GitLab, from Gerrit](#2-use-g2g-to-import-phobos-into-gitlab-from-gerrit)
3. [Configure GitLab CI](#3-configure-gitlab-ci)
4. [Explanation of Migrated Phobos Tests](#4-explanation-of-migrated-phobos-tests)
5. [Edits you should make to CI scripts](#5-edits-you-should-make-to-ci-scripts)

<br />

## 1. Create Phobos GitLab Project

Create a new GitLab project for Phobos by following the official documentation for creating a blank project:
* https://docs.gitlab.com/user/project/

<br />

## 2. Use G2G to Import Phobos into GitLab, from Gerrit

Once a blank project has been created, the next step is to import the Phobos project from Gerrit into GitLab. 

To do this, you will need proper credentials to authenticate with GitLab and enable you to push to the blank Phobos project. Here we use a Personal Access Token (PAT) as our credential. To create a PAT, follow these instructions:
* https://docs.gitlab.com/user/profile/personal_access_tokens/#create-a-personal-access-token

You also need credentials which enable you to clone the sources from Gerrit.

Once you have your credentials, use the Gerrit-to-GitLab (g2g) tool to migrate the Phobos data from Gerrit to GitLab.

```
g2g \
 --gerrit-url https://gerrit.ccc.ocre.cea.fr/gerrit \
 --gerrit-login '<GERRIT_USER>:<GERRIT_TOKEN>' \
 --gerrit-project phobos \
 --gitlab-url https://gitlab-forge.ccc.ocre.cea.fr \
 --gitlab-login '<PAT_TOKEN_NAME>:<PAT_TOKEN_VALUE>' \
 --gitlab-project <PHOBOS_GITLAB_NAMESPACE>/<PHOBOS_GITLAB_PROJECT_NAME> \
 --noproxy-gerrit \
 --noproxy-gitlab \
 --timeout 60
```

The value of `--gitlab-project`, for us, was `paratools/phobos`. Based on our discussions with you, you will use a namespace other than `paratools` to hold the project. Substitute your chosen namespace for `<PHOBOS_GITLAB_NAMESPACE>`.


<br />


## 3. Configure GitLab CI

The final step is to configure GitLab CI so that the tests formerly run in Jenkins are available in GitLab.

We have done this migration for you. To take advantage of this work, you need to do the following:
1. Cherry-pick our work into the new Phobos project
2. Update GitLab settings for the new Phobos project

<br />

### Cherry-pick our work into your new project

The Jenkins Phobos tests have been imported by us into our test project `paratools/phobos`.
* https://gitlab-forge.ccc.ocre.cea.fr/paratools/phobos

On the `paratools/phobos` master branch, there is a single commit that you need to cherry-pick into the master branch of your new Phobos project.

```
Commit ID = aeeb5b6de1c485c44cd5fa48cd94d98519af6b6d

$> git diff --name-only HEAD^
ci/run-ci.sh
gitlab-ci/.gitlab-ci.yml
gitlab-ci/code-check.sh
gitlab-ci/phobos-ci.sh
gitlab-ci/pylint.sh
gitlab-ci/utilities.sh
```

Simply cherry-pick this single commit onto the master branch of your new Phobos project.

<br />

### Update GitLab Settings for the new Phobos project

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

## 4. Explanation of Phobos Tests Migrated from Jenkins

The following tests have been migrated from Jenkins to GitLab:
1. Code Check
2. Pylint
3. RHEL-8 Tests
4. RHEL-9 Tests
5. Valgrind Tests
6. Rados Tests (Disabled in Jenkins, Disabled in GitLab)

<br />

<img width="1386" height="574" alt="Phobos Tests" src="https://github.com/user-attachments/assets/fbaca73b-2a29-40b8-92d6-72c31d75c313" />

<br />
<br />

The scripts for these tests are all stored in the `gitlab-ci` folder at the root of the phobos project and are included in commit cherry-picked as part of Step 3.

Check `gitlab-ci/gitlab-ci.yml` for the definition of each tests CI job.

The format of gitlab-ci.yml is well documented here:
* https://docs.gitlab.com/ci/yaml/

<br />
<br />

## 5. Edits you should make to CI scripts

You may need to edit some of the values from `gitlab-ci/.gitlab-ci.yml`.

### Setting the GitLab Runner Tag

The highlighted section of the screenshot below shows the part of `gitlab-ci/.gitlab-ci.yml` used to specify the tag used for the GitLab Runner which will run the CI jobs. You should either configure your runner to have the given tag, or else change the value of the tag to match you have configured for your runner.

<img width="1413" height="372" alt="phobos-gitlab-runner-tag" src="https://github.com/user-attachments/assets/9c8c06d3-49a2-4409-a98e-d50a333f23f1" />

<br />
<br />

### Setting the PCOCC environment variables

The highlighted sections of the screenshot below show the parts of `gitlab-ci/.gitlab-ci.yml` where PCOCC parameters are set.
* PCOCC_USER_DATA should point to the same cloud-init script which is used in launching the PCOCC VM for the existing Jenkins equivalent job
* PCOCC_USER_CONF_DIR should point to whatever PCOCC configuration directory contains a definition of the VM used in PCOCC_CLUSTER_DEFINITON

<img width="1413" height="271" alt="phobos pcocc settings 3" src="https://github.com/user-attachments/assets/400127da-9d71-4228-a3ee-1491ef928627" />
<img width="1413" height="639" alt="phobos pcocc settings" src="https://github.com/user-attachments/assets/90b49c99-6a79-4521-bc12-0ec0344e5f5e" />
<img width="1413" height="294" alt="phobos pcocc settings 2" src="https://github.com/user-attachments/assets/67b08acd-d3c4-475a-a09e-2e3b90c768cc" />


<br />

<br />

<br />

<br />
