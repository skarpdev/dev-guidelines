## Version control

We use [Git](https://git-scm.com/) and [Gitlab](https://about.gitlab.com/).


### Commit messages

Writing good commit messages is important, as it helps greatly when figuring out why/when something broke.

[This article](http://chris.beams.io/posts/git-commit/) lists some good rules of thumb.


### Branching strategy

The `master` branch is only for production ready code.

The `develop` branch is the main unstable branch and is typically hooked up to automatic deployment to a test environment.

When you start working on something (however small), you branch out from `develop`. Make your changes and then submit a pull/merge request against the `develop` branch.


### Branch naming

???