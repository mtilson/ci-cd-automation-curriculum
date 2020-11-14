# CI/CD automation - Curriculum

## Basic principles of CI/CD
* CI (continuous integration) is a way to fight against `merge or integration hell`
* CD (continuous delivery) makes sure that the software checked in on the mainline is always in a state that can be deployed to users
* CD (continuous deployment) makes the deployment and rollback processes fully automated

<details>
<summary>Click to expand/collapse details</summary>
  
### Motivation
* The longer development continues on a branch without merging back to the mainline, the greater the risk of multiple integration conflicts and failures when the developer branch is eventually merged back. This is called `integration hell` or `merge hell`
* When developers submit code to the repository they must first update their code to reflect the changes in the repository since they took their copy. The more changes the repository contains, the more work developers must do before submitting their own changes
* The solution to the above issues are - frequently merging back to mainline and checking that everything is in good state, meaning we could release code
* By frequently merging back and releasing we can early detect errors and eliminate inconsistencies, it allows us
  * To lower the risks of release delaying
  * To lower time and work effort of integration
  * To reduse the cost of development
  
### Practices
In practice we have to do the following
* Run automated unit tests in the developer's local environment before committing to the mainline
* Commit to the mainline continuously and frequently
* Apply the following set of quality control principles on build server after every commit to the mainline
  * Compile the code
  * Run the unit and integration tests
  * Run additional static analyses
  * Measure and profile performance
  * Extract and format documentation from the source code
  * Facilite manual QA processes
  * Report the results to the developers
</details>

## Gitflow as a way to do CI/CD
* To satisfy the above requirements we use a dedicated Git workflow (Gitflow) where all the above operations are triggered by particular events (commits, push to remote repo, merge requests, etc) in Git repository with the code being developed
* Gitflow is just usual Feature Branch Git workflow but with very specific roles assigned to different branches (individual branches used for preparing, maintaining, and recording releases) and defined interaction between the branches
* The result of using Gitflow in such a way is a process of **automation in releasing applications** which implies automation in _building, testing and deployment_ of applications

<details>
<summary>Click to expand/collapse details</summary>
  
### Specific branches
* master — Branch name for production releases - long-lived branch
* develop — Branch name for "next release" development - long-lived branch
* feature/* — Feature branches - temporary branches
* release/* — Release branches - temporary branches
* hotfix/* — Hotfix branches - temporary branches

### The Gitflow flow: Overview
1. Long-lived branches are only  *master* and *develop*
   * A *master* branch is created initially with repo
   * A *develop* branch is forked from *master* branch right after that
2. *Feature* branches are forked on local repo from *develop* branch
3. When a *feature* is complete it is pushed to remote repo where it is merged into the *develop* branch via **merge request**
   * *Feature* branches on local repo is deleted then
   * *Feature* branch on remote repo lives only till the moment the **merge request** is accepted and then it is deleted
4. A *release* branch is forked from *develop* branch and lives till the moment it is merged into *master* and *develop* branches
5. When the *release* branch is done it is merged into *develop* and *master* branches and then the *release* branch is deleted
6. If an issue in *master* branch is detected a *hotfix* branch is forked from *master* branch
7. Once the *hotfix* is complete it is merged into both *master* and *develop* branches and then the *hotfix* branch is deleted
   * If *hotfix* branch is created during the time range when *release* branch exists, then it is merged into *master* and *release* branches (in addition or insted of *develop* branch) and then the *hotfix* branch is deleted

### The Gitflow flow: Details
* Long-lived *master* branch stores the official release history
  * All commits in the *master* branch are tagged with a version number
* Long-lived  *develop* branch serves as an integration branch for features
* Temporary *feature* branches use *develop* branch as their parent branch
  * When a *feature* branch is complete, it gets merged back into *develop* branch
  * *Feature* branches should never interact directly with *master* branch
  * *Feature* branches are deleted after merged into *develop* branch
* Temporary *release* branches start the next release cycle
  * Using a dedicated branch to prepare releases makes it possible for one team to polish the current release while another team continues working on features for the next release
  * *Release* branches are forked off of *develop* branch once *develop* branch has acquired enough features for a release (or a predetermined release date is approaching)
  * No new features can be added after this point to the release - only hotfixes, documentation generation, and other release-oriented tasks should go in this *release* branch
  * Once *release* branch is ready to ship, it gets merged into *master* branch and tagged with a version number
  * After that *release* branch is merged back into *develop* branch, which may have progressed since the release was initiated. It’s important to merge back into *develop* branch because critical updates may have been added to the *release* branch and they need to be accessible to new features
  * *Release* branch is deleted after merged into *develop* branch
* Temporary *hotfix* branches are used to quickly fix bugs and patch production releases in the *master* branch
  * Having a dedicated line of development for bug fixes lets your team address issues without interrupting the rest of the workflow or waiting for the next release cycle
  * *Hotfix* branches are like *release* branches and *feature* branches except they are based on *master* branch instead of *develop* branch
  * *Hotfix* branches are the only branches that should fork directly off of *master* branch
  * As soon as the fix is complete, it should be merged into both *master* and *develop* branches (or the current *release* branch), and *master* branch should be tagged with an updated version number (indicating that it was hotfix)
  * *Hotfix* branches are deleted after merged into *develop* branch
</details>

## CI/CD applications
* **CI/CD application** is dedicated kind of application used to create and manage a process of **automation in releasing applications**
* The main purposes
  * Providing a way to create **CI/CD pipelines** - defined sets of steps to be performed as reaction on events in a repository with the code being developed
  * Managing execution of pipeline steps and configuring execution environments 
* Modern CI/CD platforms are being created with the following ideas in aims to support the mentioned purposes
  * To facilitate the creation of _pipeline specification_ (usually using `yaml`-based DSL) 
  * To universalize execution environment (usually using containerization)
* Here are some CI/CD tools initially professing the above approach
  * [GitLab CI/CD](https://docs.gitlab.com/ee/ci/pipelines/), [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/), [GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions)

<details>
<summary>Click to expand/collapse details</summary>
  
### Motivation
* We need a standardized way to define _pipeline specification_ to be able not to change this specification if we like to start using different CI/CD platforms
* We need a common and univesral way to manage different execution environments and their configurations for all the required pipeline steps - to be independent on CI/CD platforms as well

### Practices
In practice we have the following
* Dozens of different CI/CD tools and platforms from different vendors, open source and not only
* No standardized way to define _pipeline specification_
  * The most common approach is to use `yaml`-based vendor specific DSL
* As `docker` containerization has become 'de facto' standard to run any applications, CI/CD platforms tend to use `docker` as the most convenient and univesral way to execute pipeline steps
  * The most common approach is to use `docker in docker (DIND)` to be able to setup and run services necessary for mocking application environments used in development/staging/production environments
* As a result we have to choose one of the existing CI/CD platforms (and probably be coupled to its approach to define _pipeline specification_). The criteria to take into account
  * Native support for `yaml`-based _pipeline specifications_ as the most common way to manage pipelines
  * Native support for `docker in docker` as a way to execute pipeline steps 
* We will use the following tools to provide examples
  * **GitLab CI/CD**
  * **Bitbucket Pipelines**
  * **GitHub Actions**
</details>

## CI/CD pipeline
* **CI/CD pipeline** is a defined set of steps to be performed as reaction to events in a repository with the code being developed
* To define pipeline we have to
  * determine a set of Git repo events (according to the Gitflow) the CI/CD application is going to watch for and trigger processing steps
  * specify steps executed in responce to the triggered events
* Usually **CI/CD applications** provide ability to configure **CI/CD pipeline** only for events in remote Git repo
* To watch for events and execute pipeline steps in local Git repo it is possible to use **Git hooks**

<details>
<summary>Click to expand/collapse details</summary>
  
### Motivation
* We have to be able to execute the same pipeline steps both for remote (via configuring **CI/CD applications**) and for local (via configuring **Git hooks**) Git repo events
* We have to be able to parameterize pipeline steps depending on repo events
 
### Practices
* Pipeline steps are defined as self-sufficient _scripts_ as we can use them both in **CI/CD applications** and in **Git hooks**
* Pipeline step parameterization is done with help of environment variables
  * _Scripts_ behavior depends on conditions derived from transferred environment variables
* **Git hooks** are managed by [pre-commit](https://pre-commit.com) framework

### Events within local Git repo (processed by Git hooks)
| Repo event | Run in response | Deployment environment | Details |
|:-----------|:----------------|:-----------------------|:--------|
| 1. *Develop* branch is checked out to a new *feature* branch | | | |
| 2. Remote *develop* branch is merged into local *develop* branch | | | |
| 3. *Develop* branch is merged into *feature* branch <br> 4. Changes are commited into *feature* branch | Linting / Compiling | | |
| | Unit tests | | Dependencies required to run Unit tests are mocked |
| 5. *Feature* branch commit is tagged | Building | | Container images are built and stored locally |
| | Deployment | _Local temporal_ | _Local temporal_ deployment environments are created locally with help of `docker compose` or `minikube/kind/microk8s/...` |
| | Integration tests | _Local temporal_ | Required dependent services are created locally within deployment environment |
| | QA tests | _Local temporal_ | _Local temporal_<sup id="a01">[1](#f01)</sup> deployment environment can be configured to stay alive after pipeline execution for proceeding with manual QA processes |

* <b id="f01">1:</b> For QA interaction with _local temporal_ deployment environments it can be identified by specific feature branch name and commit ID (like `feature-ABC-XXXXXXX`, where `ABC` is feature name and `XXXXXXX` is 7 digits of commit ID) [↩](#a01)

### Events within remote repo (processed by CI/CD applications) 

| Repo event | Run in response | Deployment environment | Details |
|:-----------|:----------------|:-----------------------|:--------|
| 1. Local *feature* branch is pushed to remote *feature* branch and **merge request** into *develop* branch is created | Linting / Compiling | | |
| | Unit tests | | Dependencies required to run Unit tests are mocked |
| | Building | | Container images are built and stored in container registry |
| | Deployment | _Development temporal_ | _Development temporal_ deployment environments are created with help of `DIND` services (within CI/CD execution environment), `docker compose`, or `k8s` |
| | Integration tests | _Development temporal_ | Required dependencies can be created within deployment environment locally or using external services |
| | QA tests | _Development temporal_ | _Development temporal_<sup id="a02">[2](#f02)</sup> deployment environment are configured to stay alive after pipeline execution for proceeding with manual QA processes |
| 2. **Merge request** is rejected | Cleanup | | Corresponding _development temporal_ environment is deleted |
| 3. **Merge request** is accepted (*feature* branch is merged into *develop* branch) | Cleanup | | Corresponding _development temporal_ environment is deleted |
| | Building | | Container images are built and stored in container registry |
| | Deployment | _Stage_ | _Stage_ deployment environment is created with help of `docker compose` or `k8s` |
| | Integration tests | _Stage_ | Required dependencies to be created within deployment environment using the same or similar services as production environment |
| | QA tests | _Stage_ | _Stage_<sup id="a03">[3](#f03)</sup> deployment environment is permanently available |
| 4. *Release* branch is forked from *develop* branch | Deployment | _Pre-production_ | _Pre-production_ deployment environment is created with help of `docker compose` or `k8s` |
| | Integration tests | _Pre-production_ | — Required dependencies to be created within deployment environment using the same services as production environment <br> — Copy of production DB is used <br> — Possible production environment variables are used |
| | QA tests | _Pre-production_ | _Pre-production_<sup id="a04">[4](#f04)</sup> deployment environment is permanently available |
| 5. *Release* branch is merged into *master* branch | Deployment | _Production_ | _Production_ deployment environment is created with help of `docker compose` or `k8s` and is permanently<sup id="a05">[5](#f05)</sup> available |
| 6. *Release* branch is merged into *develop* branch | Building / Deployment / Integration tests / QA tests | _Stage_ | All steps are the same as for event **3.** |
| 7. Changes are commited into *hotfix* branch <br> 8. *Hotfix* branch is merged into *release* branch | Deployment / Integration tests / QA tests | _Pre-production_ | All steps are the same as for event **4.** |
| 9. *Hotfix* branch is merged into *master* branch | Deployment | _Production_ | All steps are the same as for event **5.** |
| 10. *Hotfix* branch is merged into *develop* branch | Building / Deployment / Integration tests / QA tests | _Stage_ | All steps are the same as for event **3.** |

* <b id="f02">2:</b> For QA interaction with _development temporal_ deployment environments it can be identified by specific feature branch name and commit ID (like `feature-ABC-XXXXXXX`, where `ABC` is feature name and `XXXXXXX` is 7 digits of commit ID) [↩](#a02)
* <b id="f03">3:</b> For QA interaction with _stage_ deployment environments it is identified by its own constant ID (like `stage`) [↩](#a03)
* <b id="f04">4:</b> For QA interaction with _pre-production_ deployment environments it is identified by its own constant ID (like `preprod`) [↩](#a04)
* <b id="f05">5:</b> For interaction with _production_ deployment environments it is identified by its own constant ID (like `prod` or just application public  domain name) [↩](#a05)
</details>

## Tooling
Based on the processed described above we have to deal with the following tools
* Git - as a single tool to operate software development life cycle
* CI/CD pipline platforms — precreated integrations and different ways to automate workflow
* Containerization and CI/CD automation

<details>
<summary>Click to expand/collapse details</summary>
  
### Git and Software Development Life Cycle
* Distributed VCS: local, remote repo's
* Basic commands: init, clone, add, commit, branch, merge, pull, push
* Basic flow:
  1. check out to new branch: branch, checkout,
  1. local change cycle: add, commit, merge
  1. merge changes to remote: push, (pull request), merge

### CI/CD pipline platforms
* GitLab CI/CD and GitLab Workflow
* Bitbucket Pipelines and Bitbucket Pipes
* GitHub Actions

### Containerization (based on Docker) and CI/CD automation
* **Building** applications as Docker images
  * Docker images — portable snapshots of your environment and its dependencies
  * Dockerfile — a way to declare how a docker image should be build
  * Docker multi-stage builds - lean containers
    * TODO: more about an idea of 'Using multiple Dockerfiles in separate directories for different applications to help share common files'
  * Caching — each declaration within a Dockerfile is cached when it's first built
  * Docker registry - a way to store Docker images
  * Production-ready Docker images
    * `docker-entrypoint.sh`
* **Running** applications as Docker containers
  * Docker containers - an universal approach to running applications
* **CI/CD automation**
  * Using containers to do integration tests  
  * Using containers in production — orchestration
    * Communication issues and container networking
    * Docker Compose
    * Kubernetes
* **References**
  * [Concept of containers](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)
  * [What's a Linux container?](https://www.redhat.com/en/topics/containers/whats-a-linux-container)
  * [Docker multi-stage builds](https://docs.docker.com/engine/userguide/eng-image/multistage-build/#name-your-build-stages)
  * [Docker networking](https://docs.docker.com/engine/userguide/networking/)
  * TODO: more
</details>

## Practical principles
* Mono-repo VS multi-repos
* Pipeline specifics

<details>
<summary>Click to expand/collapse details</summary>
  
### Mono-repo VS multi-repos
* Mono-repo — no issues to share `common parameters`
* Multi-repos - there are different ways to share `common parameters`
  * git shared-repo
  * GitLab `include`
  * TODO: more about different approaches

### Pipeline specifics
* Create and publich (store in image registry) `build image` - to be used to create apps images
  * Use multi-sgaing to separate building and execution environments resulting to creation of lean app images 
  * Tagging is a way to differentiate versions of your application
* Create and publich `deploy image` - to be used to deploy apps to different environments
  * Use parameterization to run app with different configuration in different environments 
* TODO: more about other specifics

### TODO: more specifics here
</details>
