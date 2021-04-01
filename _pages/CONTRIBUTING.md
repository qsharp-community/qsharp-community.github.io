---
permalink: /contributing/
title: Contributing to Q# Community
toc: true
author_profile: true
---

This community encourages feedback and contributions.
Thank you for your interest in making the Q# Community better!
There are several ways you can get involved.

## Finding tasks you can help with

Looking for something to work on?
Issues marked ``good first issue`` on all community repos are a good place to start.
If you're interested in working on a fix, leave a comment to let everyone know and to help avoid duplicated effort from others.

You can also take a look at our [community roadmap](https://github.com/qsharp-community/roadmap) to see if there is any of our more general community goals you would be interested in helping with.

## Adding a new project to the community

Are you someone who has a project they want to contribute to the Q# community organization?
There are a few main benefits to doing so:

- Make it easier for new contributors to find your project 🎉
- Crowd source maintaining and developing with others familiar with Q#

### Steps to add a new project
1. Send an email introducing you and your project to [projects@qsharp.community](mailto:projects@qsharp.community) (with a link to the current repo if possible) for the organization maintainers to review. 
2. Then at least two maintainers will review the project to make sure it is a good fit. We strive to get feedback on this in a timely manner.
3. If the project is a good fit, next is to start a new repo or transfer the ownership of the previous repo to the organization. Any original contributors will be made admin on the repo under the org and if the licence is [OSI approved](https://opensource.org/licenses), it can maintain all original licensing.
4. 🎊**Party!**🎉 Now if you like you can write a blog post about your project and we can host/cross-post it to our site here so people can read first hand from you about your project! Also consider publishing your work to Nuget or Myget!

### Licencing repositories in the community organization

Each repository when contributed to the organization can decide what [OSI approved](https://opensource.org/licenses) licence it uses.
This is an open source community and the best way to help others learn and continue to build amazing projects is to keep making projects open source.
The community generally suggests MIT licence, as most open source .NET projects do the same.

## Reporting issues and suggesting new features

If something about one of the community projects is not working properly, please file an issue of the corresponding repository.
If you have ideas for new features of the community, please post in the #ask-the-admin channel on the [Discord](https://discord.qsharp.community) or [file an issue](https://github.com/qsharp-community/qsharp-community.github.io/issues/new/choose) on our main repo.
Remember that all community interactions must abide by the [Code of Conduct](CODE_OF_CONDUCT.md).

## Contributions we accept

We welcome your contributions to Q# community projects!🎂
Some general guidelines:

* **DO** follow the [Q# style guide](https://docs.microsoft.com/en-us/quantum/contributing/style-guide?view=qsharp-preview), and keep code changes as small as possible.
* **DO** include corresponding tests whenever possible.
* **DO** check for additional occurrences of the same problem in other parts of the codebase before submitting your PR.
* **DO** [link the issue](https://github.com/blog/957-introducing-issue-mentions) you are addressing in the pull request.
* **DO** write a good description for your pull request. More detail is better. Describe *why* the change is being made and *why* you have chosen a particular solution. Describe any manual testing you performed to validate your change.

## Making changes to the code

### Style guidelines

The Q# language has a [style guide](https://docs.microsoft.com/en-us/quantum/contributing/style-guide?view=qsharp-preview), please use this for Q# code projects.
For projects that involve other languages, please use consistent styling.

### Testing and Packaging

Your change should include tests to verify new functionality wherever possible.
> Coming Soon: instructions on how to setup GitHub Actions for Q# projects

## Review Process

After submitting a pull request, members of the project will review your code.
We will assign the request to an appropriate reviewer.
Any member of the community may participate in the review, but at least one member of the project will ultimately approve
the request.
