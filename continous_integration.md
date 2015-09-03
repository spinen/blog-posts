Continous Integration (CI) and Continous Delivery (CD) are natural counterparts of the Agile process. At Spinen we started working on a  model of Continous Integration about 2 years ago. We are a small team (about 7 people) and our CI model has evolved over time. About 1 year ago we brought on some new graduates. Bringing on new people really forced us to standardize some practices we had been trying. It also helped us identify some (figurative) landmines that we had all simply known to step around up to that point.

Our CI model is built from the [git-flow](http://git-flow.com) model of project development. Usually any project is cloned from our working 'base' for that type of project. The new project therefore starts with identical develop and master branches. From that starting point our project development then goes like this:

1. Developer A - let's call him Bobba - clones the project and initializes git flow locally with `git flow init -d`
2. Bobba then creates a feature branch with `git flow feature start add-widget` and begins work on his widget.
3. Bobba finishes his widget and runs tests locally.
4. Bobba pushes `feature/add-widget` to the central git repository. (this is where we stray slightly from git-flow orthodoxy)
5. Bobba's push automatically runs all tests on our [jenkins](http://jenkins.com) server
6. Bobba makes a Merge Request (MR) from his branch (`feature/add-widget`) to `develop` and assigns it to another developer - let's call him Jango. Because jenkins ran all tests on the feature push, we are able to see the test status of feature before we merge.  This allows the merge reviewer to focus on higher level questions, rather than bothering about linting, etc.

```
We have had great success with making MRs from junior -> senior and from senior -> junior. However describing our thought process on that, and code review in general, is a separate blog post.
```
7. Jango approves - `feature/add-widget` is merged into `develop`. This prompts a series of actions by jenkins:
	a. Run all tests again
	b. Create an artifact (a zip file in this case) of everything needed to deploy the code, including all dependencies.
	c. Deploy the code to our development environment.

Because of this process code in our `develop` branch stays relatively stable. But you've probably noticed that the widget Bobba added is still not in production, where the customer is desparately waiting for it. The process for it to get there is also simple:

1. Jango checks out the `develop` branch of the project with all recent changes.
2. Jango starts a release branch with the command `git flow release start X.Y.ZZ`
3. Jango pushes the release branch to Jenkins which automatically prompts Jenkins to:
	a. Test the relase branch.
	b. Create a zip file artifact of the code with all dependencies.
	c. Deploy the code to our QandA environment

The QandA site is a publicly accessible site deployed to a production-like environment. This allows the customer to mess around with the new widget and find all the bugs that we haven't already found.  At this point any work done on the `release/X.Y.ZZ` branch should be bugfixes to make the promised widget work. If the customer comes up with new ideas, for a new feature, those changes should not be added to the current release.

Assuming the customer loves the widget and is ready to push it to production:

1. Jango merges any changes to `release/X.Y.ZZ` locally. He then give the command `git flow release finish X.Y.ZZ` This merges the current relase branch into the `develop` and `master` branches.
2. Jango pushes `develop` and `master` to the central git repo.
3. Jenkins follows the same process to deploy the `develop` branch to the development environment and the `master` branch to the production environment.