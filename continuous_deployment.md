Continuous Integration (CI) and Continuous Deployment (CD) are popular software development buzzwords that mean a lot of things to a lot of people. At SPINEN we really embraced the concepts of CI/CD about 2 years ago, and have gone through several iterations of the process. We are not perfect now, but we are much better now than we were when we started (Continuous Improvement.)

Our deployment process maps environments to the [git-flow](https://github.com/nvie/gitflow) cycle. It's best shown by example:

#### Deploying to the Development environment

1. Developer A - let's call him Robin - checks out a project and starts a new feature: `feature/add-widget`.

2. Robin finishes the feature and runs all tests locally.

3. Robin pushes his feature branch to the central git repository. This action prompts all tests to run in [Jenkins](http://jenkins-ci.org) via web hook.

4. Robin makes a merge request (MR) to another developer, let's call him Batman (we usually assign MRs Junior -> Senior and Senior -> Junior.) This action prompts all tests to run on Jenkins. The test status of the branch shows up in the merge request.

5. Batman accepts the MR and branch `feature/add-widget` is merged into `develop`. 

(Since the developer has his branch merged by the git tool, he must manually delete the branch locally. We hardly every use `git flow feature finish`)

---- Continuous deployment of Development starts here-----------

An accepted merge request is

All tests run on Develop, a build artifact is created. (composer install, etc.)

Jenkins copies the artifact to a target server directory named /var/www/project/BUILD_ID

Do all commands to prepare the site (migrate the DB, etc)

Move the symlink /var/www/project/current to point to /var/www/project/BUILD_ID

New feature is deployed to Development environment!

-------- deploying to Qanda ---------

Developer B starts a release branch with git flow release/0.2.3

Push the release branch to Jenkins 

All tests run in Jenkins -> Build artifact is created.

deployment process is exactly the same as development, but to Qanda.

------ deploying to Production -----

Merge release into master. 

Push master to git repo.

deployment process is identical