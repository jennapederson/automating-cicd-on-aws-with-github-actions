+++
title = "3. Create a CodeBuild Project"
chapter = true
weight = 50
+++

# Create a CodeBuild Project

AWS CodeBuild is used to execute our application tests and provide the status of these tests to GitHub. If the tests do not pass, CodeBuild marks the build as Failed and this status is reported to GitHub so our broken application code won't be deployed to the container service. In our architecture, we rely on webhooks that are automatically created by CodeBuild to trigger a build of our application code every time there is a commit to the main branch of the GitHub repository.

Let's create the CodeBuild project!

The following steps will be completed in the AWS Console.

## 1. Configure Project

Navigate to [AWS CodeBuild](https://console.aws.amazon.com/codesuite/codebuild/start?region=us-east-1) and select "Create project"

![](/images/code-build-create.png)

Under Project Configuration, for "Project name", enter `ecs-devops-sandbox`.

![](/images/code-build-project-config.png)

## 2. Configure Source

Next, you'll configure the Source provider, which source control repository to pull from and trigger builds.

1. For "Source Provider", select "GitHub".
1. For "Repository", select "Connect using OAuth" and then select "Connect to GitHub"
1. If you haven't configured a GitHub connection yet, you'll be presented with a pop-out window to connect to GitHub. At the pop-out window, log into the GitHub account that owns the repository you wish to connect.
1. For "Repository", select "Repository in my GitHub account".
1. For "GitHub repository", select the repository you configured earlier, `demo-github-actions`.
1. Expand "Additional configuration". For "Build Status – optional", check the box to "Report build statuses to source provider when your builds start and finish".

![](/images/code-build-source.png)

## 3. Configure Webhooks

By configuring a webhook, our build will be triggered every time there's a code change pushed to this repository in GitHub.

Under "Primary source webhook events", for "Webhook – optional", check the box to "Rebuild every time a code change is pushed to this repository".

![](/images/code-build-webhook.png)

## 4. Configure Environment

1. For "Environment image", select the default, "Managed image".
1. For "Operating system", select "Ubuntu".
1. For "Runtime", select "Standard".
1. For "Image", select "aws/codeBuild/standard:4.0".

![](/images/code-build-environment.png)

Leave remaining default values and then select "Create build project". Now that we have our CodeBuild project setup, we need to create a GitHub action.

![](/images/code-build-success.png)