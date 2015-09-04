Continuous Integration (CI) and Continuous Delivery (CD) are natural counterparts of the Agile process. At Spinen we started working on a  model of Continuous Integration about 2 years ago. We are a small team (about 7 people) and our CI model has evolved over time. About 1 year ago we brought on some new graduates. Bringing on new people really forced us to standardize some practices we had been trying. It also helped us identify some (figurative) land mines that we had all simply known to step around up to that point.

Our CI model is built from the [git-flow](https://github.com/nvie/gitflow) model of project development. Usually any project is cloned from our working 'base' for that type of project. The new project therefore starts with identical develop and master branches. From that starting point our project development then goes like this:

1. Developer A - let's call him Robin - clones the project and initializes git flow locally with `git flow init -d`
2. Robin then creates a feature branch with `git flow feature start add-widget` and begins work on his widget.
3. Robin finishes his widget and runs tests locally.
4. Robin pushes `feature/add-widget` to the central git repository. (this is where we stray slightly from git-flow orthodoxy)
5. Robin's push automatically runs all tests on our [Jenkins](http://jenkins-ci.org) server
6. Robin makes a Merge Request (MR) from his branch (`feature/add-widget`) to `develop` and assigns it to another developer - let's call him Batman. Because Jenkins ran all tests on the feature push, we are able to see the test status of feature before we merge.  This allows the merge reviewer to focus on higher level questions, rather than bothering about linting, etc.
7. Batman approves - `feature/add-widget` is merged into `develop`. This prompts a series of actions by Jenkins:
	a. Run all tests again
	b. Create an artifact (a zip file in this case) of everything needed to deploy the code, including all dependencies.
	c. Deploy the code to our development environment.
	
```
We have had great success with making MRs from junior -> senior and from senior -> junior. However describing our thought process on that, and code review in general, is a separate blog post.
```

Because of this process code in our `develop` branch stays relatively stable. But you've probably noticed that the widget Robin added is still not in production, where the customer is desperately waiting for it. The process for it to get there is also simple:

1. Batman checks out the `develop` branch of the project with all recent changes.
2. Batman starts a release branch with the command `git flow release start X.Y.ZZ`
3. Batman pushes the release branch to Jenkins which automatically prompts Jenkins to:
	a. Test the release branch.
	b. Create a zip file artifact of the code with all dependencies.
	c. Deploy the code to our QandA environment

The QandA site is a publicly accessible site deployed to a production-like environment. This allows the customer to mess around with the new widget and find all the bugs that we haven't already found.  At this point any work done on the `release/X.Y.ZZ` branch should be bug fixes to make the promised widget work. If the customer comes up with new ideas, for a new feature, those changes should not be added to the current release.

Assuming the customer loves the widget and is ready to push it to production:

1. Batman merges any changes to `release/X.Y.ZZ` locally. He then give the command `git flow release finish X.Y.ZZ` This merges the current release branch into the `develop` and `master` branches.
2. Batman pushes `develop` and `master` to the central git repo.
3. Jenkins follows the same process to deploy the `develop` branch to the development environment and the `master` branch to the production environment.

That's the whole process. It's as "continuous" as we can stand it right now. I've tried hard here to focus on the process of writing code and merging code while ignoring many other things that are worth diving into, like how we actually deploy the code.

The most important thing to me: it works for us right now, enabling us to effectively write, test and deploy code in a timely manner. From this position we can iterate to make our process even better.