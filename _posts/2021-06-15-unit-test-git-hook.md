---
layout: post
title:  "Running unit test automatically before pushing"
date:   2021-06-15 16:02:32 -0400
categories: git
---

Something that I have been using lately is a git pre-push hook to build and run our unit test suite before pushing. I like that I can ensure that I do not push any commits that break the build or unit tests and that I get fast feedback.

[Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) allow for you to run scripts pre or post important workflow events (commit, push, rebase, etc). Hooks are stored in \.git\hooks and contain a bunch of samples. For my case, I want to run my unit test project, check if all tests passed, and prevent the push if any failed.

To do this I need to create `pre-push` script. In this script I use `dotnet test` to build and run all test projects in the dotnet folder and verify that all tests pass. If anything fails, I exit with a error return code of 1. This prevents the push from continuing. 
The folder `dotnet test` points to needs to be updated to point to either the location of your solution, or the unit test project (see `dotnet test` documentation for more details). Also note that the script file _has no extension_.

I'll be the first to admit that I don't write many unix shell scripts so don't expect anything pretty and certainly there are better ways to locate the unit test project, but here is what I'm using for my project. 

File `.git/hooks/pre-push`
```sh
#!/bin/sh
echo "Running unit tests before pushing"
dotnet test "./dotnet/"

if [ $? != 0 ]; then
	echo "Aborting push, unit tests failed."
	exit 1
fi
```