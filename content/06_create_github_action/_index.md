+++
title = "Create GitHub Action"
chapter = true
weight = 60
+++

# Create a GitHub Action

AWS provides a starter GitHub workflow that takes advantage of the AWS open-source GitHub Actions to build and deploy containers on ECS for each commit to the main branch of a repository.

1. Go to your repository on GitHub.
2. Go to the Actions tab
3. Find "Deploy to Amazon ECS" and select "Set up this workflow"
4. Following the instructions outlined in the starter workflow file for using this workflow with our application, we will reference the ECR repository, ECS task definition, ECS cluster, and ECS service that we created previously.
  1. Replace `ECR_REPOSITORY` with  `ecs-devops-sandbox-repository`
  2. Set the value of aws-region to `us-east-1` (or whatever region is shown in the upper right of the AWS Console)
  3. Set the value of service to `ecs-devops-sandbox-service`
  4. Set the value of container-name to `ecs-devops-sandbox`

  5. Select Start commit
  6. Enter a commit message
  7. Select Commit new file

## Create IAM user with programmatic access

TBD - Create an IAM user and note the AWS ACCESS KEY ID and the AWS SECRET ACCESS KEY for later.

## Setup Secrets for GitHub Action

To deploy to ECR we need to give GitHub permission to speak to AWS. To do that we add secrets to the repository.

Secrets are environment variables that are encrypted and can be used by your GitHub action workflow to access external resources like AWS.

1. Go to the Settings tab of your GitHub repository
2. Select Secrets
3. Select New repository secret
4. Set Name to `AWS_ACCESS_KEY_ID`
5. Set value to the access key id you copied from the previous section.
6. Set Name to `AWS_SECRET_ACCESS_KEY`
7. Set the value to the secret access key you copied from the previous section.

You should now have two secrets like this:

TBD IMAGE

## Update the GitHub action to trigger on push to main

We now need to change the action to trigger the workflow whenever a change is pushed to the main branch.

Pull your changes from GitHub to get the aws.yml workflow file:

```
git pull
```

In your editor, open `.github/workflows/aws.yml`. Instead of triggering the workflow when a release is created, we want to change it to when changes are pushed to the main branch.

Replace these lines:

```
on:
  release:
    types: [created]
```

with these lines:

```
on:
  push:
    branches:
      - main
```

## Add Step to Check Build Commit Status

Now we need to add a step to workflow to check the build commit status. If the CodeBuild workflow is unsuccessful, the workflow exists.

Add the following as the first step under steps to your `.github/workflows/aws.yml` file:

{{%expand "View the `.github/workflows/aws.yml` file" %}}

```
TBD
```

{{% /expand%}}

Commit and push these changes to GitHub.

```
git add .
git commit -m "Update workflow to run on push to main branch"
git push origin
```