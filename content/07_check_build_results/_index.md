+++
title = "5. Check the build results"
chapter = true
weight = 70
+++

# Check the build results

When this code is pushed to the main branch of the repository, we can see that CodeBuild is automatically invoked via webhook and executes the application tests. And it looks like our first build is successful!

![](/images/code-build-build-success.png)

We can drill in further to that CodeBuild build project and look at individual build statuses. Select the "Name" of the build to see more detail about the build runs.

![](/images/code-build-build-success-detail.png)

Then you can drill into that specific run by selecting the "Build run" link.

![](/images/code-build-build-success-run.png)

After the CodeBuild build runs, then it reports the successful or unsuccessful status to GitHub.

TBD IMAGE - GitHub success

But what happens if our build is failing? Let's see how to fix it in the next step!