+++
title = "6. Fix a broken build (optional)"
chapter = true
weight = 80
+++

# Fix a broken build

With the AWS CodeBuild project and GitHub workflow in place, we can add new features to our application that automatically deploy to Amazon ECS if all tests pass. If our application is broken, it does not deploy. But how do we fix a broken build?

Back in the Cloud9 development environment, open the `app.py` file and change this line:

```
return "".join(reversed(random_string))
```

to

```
return "Broken unit test!"
```

This change to the application code (app.py) will break the unit test.

Commit this change:

```
git add . && git commit -m "Break the unit test"
```

And push to GitHub:

```
git push origin
```

First, let's take a look at the CodeBuild build run to see if it was successful.

![](/images/code-build-build-failure.png)

There was a failure, as we expected. We can drill into this to see the build logs to debug the issue.

![](/images/code-build-build-failure-detail-1.png)

![](/images/code-build-build-failure-detail-2.png)

The GitHub workflow also detects the status of the commit to be a failure or success, and the remaining steps in the workflow successfully execute the deployment of the application to ECS.

![](/images/github-failure.png)

Because our GitHub workflow detected the status of the commit to be a failure, the workflow did not proceed past the “Check commit status” step.

![](/images/github-failure-detail.png)

If we revert our change and push the fix to GitHub again, we can then see that it's now successful!

![](/images/github-build-fixed.png)

Finally, we can see that the [ECS Task Definition](https://console.aws.amazon.com/ecs/home?region=us-east-1#/taskDefinitions/ecs-devops-sandbox-task-definition/status/ACTIVE) has been updated to reflect the new application container image. Note that the Git commit hash is used as the container image tag.

![](/images/ecs-task-definition-1.png)

![](/images/ecs-task-definition-2.png)

Now that you know how to set up a CI/CD workflow with Amazon CodeBuild to build and test your app, and GitHub Actions to deploy it to a container on Amazon ECS, it's time to clean up your AWS resources.

{{% notice warning %}}
If you completed this workshop in your own AWS account, do NOT skip the next step! The AWS resources you created today need to be removed from your account so that you don't get billed for any usage beyond the AWS Free Tier.
{{% /notice %}}