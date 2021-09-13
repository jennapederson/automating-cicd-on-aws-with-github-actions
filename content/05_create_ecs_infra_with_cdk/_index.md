+++
title = "3. Create a CodeBuild Project"
chapter = true
weight = 50
+++

# Create an AWS CodeBuild Project

AWS CodeBuild is used to execute our application tests and provide the status of these tests to GitHub. If the tests do not pass, CodeBuild marks the build as Failed and this status is reported to GitHub. In our architecture, we rely on webhooks that are automatically created by CodeBuild to trigger a build of our application code every time there is a commit to the main branch of the GitHub repository.

1. Navigate to AWS CodeBuild and select "Create build project"
2. Under Project Configuration, for Project name, enter ecs-devops-sandbox
3. Under Source, for Source Provider, select GitHub
4. Under Source, for Repository, select Connect using OAuth and select Connect to GitHub
5. At the pop-out window, log into the GitHub account that owns the repository you wish to
use
6. Under Source, for Repository, select "Repository in my GitHub account"
7. Under Source, for GitHub repository, select the repository you configured earlier
8. Under Source – Additional configuration, for Build Status – optional, check the box to "Report build statuses to source provider when your builds start and finish"
9. Under Primary source webhook events, for Webhook – optional, check the box to "Rebuild every time a code change is pushed to this repository"
10. Under Environment, for Environment image, select Managed image
11. Under Environment, for Operating system, select Ubuntu
12. Under Environment, for Runtime, select Standard
13. Under Environment, for Image, select aws/AWS CodeBuild/standard:4.0
14. Leave remaining default values and select Create build project