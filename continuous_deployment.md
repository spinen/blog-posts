Continuous Integration (CI) and Continuous Deployment (CD) are popular software development buzzwords that mean a lot of things to a lot of people. At SPINEN we really embraced the concepts of CI/CD about 2 years ago, and have gone through several iterations of the process. Our process is not perfect now, but we have improved it a great deal since we started (Continuous Improvement.)

Our deployment process maps environments to the [git-flow](https://github.com/nvie/gitflow) cycle. It's best shown by example:

#### Preparing the target node

Naturally the target node to which we intend to deploy the code has to have the necessary packages and tools to be able to run everything we want. We use [Chef](http://www.chef.io) as our primary configuration management tool, not only to install tools and packages, but also to set up our desired directory structure for the application. If we call our example project `virtual-batcave`, that directory structure looks like this:

1. Create directory `/var/www/virtual-batcave`
2. Create file `/var/www/virtual-batcave/.env` to hold all the relevant environment variables for the application.
3. Create directory `/data/virtual-batcave`
4. Create symlink `/var/www/virtual-batcave` pointing to--> `/data/virtual-batcave`
5. Configure the web server root directory to be `/var/www/virtual-batcave/current` (a directory that doesn't exist yet)

#### Merging a feature into the `develop` branch

1. Developer A - let's call him Robin - checks out a project and starts a new feature: `feature/add-widget`.

2. Robin finishes the feature and runs all tests locally.

3. Robin pushes his feature branch to the central git repository. This action prompts all tests to run in [Jenkins](http://jenkins-ci.org) via webhook.

4. Robin makes a merge request (MR) to another developer, let's call him Batman (we usually assign MRs from Junior -> Senior and Senior -> Junior.) This action prompts all tests to run on Jenkins. The test status of the branch shows up in the merge request.

5. Batman accepts the MR and branch `feature/add-widget` is merged into `develop`. 

(Since the developer has his branch merged by the git tool, he must manually delete the branch locally. We hardly every use `git flow feature finish`)

6. An accepted merge request is a commit to the `develop` branch, which prompts a webhook from our central git repository to Jenkins. Prompting
	a. Jenkins runs all tests again against the `develop` branch.
	b. Installs all dependencies to the local workspace. (We develop with the Laravel framework, so this generally involves [Composer](https://getcomposer.org/) and [npm](https://www.npmjs.com/))
	c. Create a zip file with **_all files necessary for deployment._**

#### Deploying the code artifact

1. Jenkins copies the artifact (with scp) to a target server directory named `/var/www/virtual-batcave/BUILD_ID`. BUILD_ID is an environment variable available in the Jenkins workspace

3. Create a symlink pointing `/var/www/virtual-batcave/BUILD_ID/.env` to--> `/var/www/virtual-batcave/.env`.

4. Create a symlink pointing `/var/www/virtual-batcave/BUILD_ID/storage` to--> `/var/www/virtual-batcave/storage`.

5. Do all commands to prepare the site (migrate the DB, etc)

4. Move the symlink `/var/www/virtual-batcave/current` to point to-->`/var/www/virtual-batcave/BUILD_ID`

New feature is deployed to the Development environment!

#### Conclusion

As we mentioned earlier. We map our environments to the git-flow model of branching merging. The `develop` branch of code is automatically deployed to the development environment, which is an environment only accessible inside our network. We deploy our `release` branches to a QandA environment, an environment that is identical to production and _is_ accessible to the world. Finally we deploy our `master` branch of code to the live production site. In all cases the deployment is prompted automatically with a push to the specific branch, and the deployment process is identical.

This deployment process is not perfect, by any means. In fact, looking through these steps almost every one presents opportunities for optimization. However one important principle here is: **_it works_**. That give us the ability to build and deploy code quickly and consistently while we improve the process.


