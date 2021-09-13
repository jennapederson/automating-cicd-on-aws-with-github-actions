+++
title = "5. Check the build results"
chapter = true
weight = 70
+++

# Check the build results

When this code is pushed to the main branch of the repository, we can see that CodeBuild is automatically invoked via webhook and executes the application tests. And it looks like our first test failed!

TBD IMAGE - CodeBuild

CodeBuild then reports the unsuccessful test execution to GitHub.

TBD IMAGE - GitHub failure

Because our GitHub workflow detected the status of the commit to be a failure, the workflow did not proceed past the “Check commit status” step.

TBD IMAGE - failed job on GitHub