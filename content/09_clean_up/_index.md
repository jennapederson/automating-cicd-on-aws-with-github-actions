+++
title = "7. Clean Up"
chapter = true
weight = 90
+++

# Clean Up Resources

{{% notice note %}}
If you are participating in this workshop as part of an event with a temporary AWS account, you can skip this step. The AWS resources you used will be removed after the workshop ends.
{{% /notice %}}

After the workshop ends, you'll want to remove the cloud resources we used in this workshop so that you don't get charged for unused resources.

## 1. Destroy the EcsDevopsSandboxCdkStack stack

In your Cloud9 environment terminal, navigate to the `ecs-devops-sandbox-cdk` directory:

```
cd ~/environment/ecs-devops-sandbox-cdk
```

Then destroy your CDK infrastructure:

```
cdk destroy
```

Confirm with a `y` that you want to delete the EcsDevopsSandboxCdkStack.

## 2. Delete the Cloud9 environment

In the AWS Console, navigate to the Cloud9 service.

![](/images/nav-cloud9.png)

Select the Delete button to delete the environment and confirm by typing the confirmation text.

![](/images/delete-cloud9.png)

## 3. Delete CodeBuild project

In the AWS Console, navigate to the CodeBuild service.

![](/images/nav-codebuild.png)

Select the project and then select the Delete build project button to delete the CodeBuild project. Confirm by typing the confirmation text.

![](/images/delete-codebuild.png)

## 4. Delete the IAM user

In the AWS Console, navigate to the IAM service.

![](/images/nav-iam.png)


Select Users from the menu and select the `github-actions-user` to delete. Select the Delete button to delete the IAM user. Confirm by typing the confirmation text.

![](/images/delete-iam-user.png)
