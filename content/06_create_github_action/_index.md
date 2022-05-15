+++
title = "4. Create GitHub Action"
chapter = true
weight = 60
+++

# Create a GitHub Action

AWS provides a starter GitHub workflow that takes advantage of the AWS open-source GitHub Actions to build and deploy containers on ECS for each commit to the main branch of a repository.

## 1. Setup GitHub Action

Go to your `demo-github-actions` repository on GitHub. On the Actions tab, find "Deploy to Amazon ECS" and select "Configure".

![](/images/gh-action-choose-workflow-1.png)

![](/images/gh-action-choose-workflow-2.png)

The instructions are outlined in detail at the top of the starter workflow file for using this workflow with your application. We will reference the ECR repository, ECS task definition, ECS cluster, and ECS service that we created previously. Here are the specific steps you'll need to complete:

  1. Set the value of the `AWS_REGION` environment variable with `us-east-1` (or whatever region is shown in the upper right of the AWS Console)
  1. Set the value for the `ECR_REPOSITORY` environment variable with  `ecs-devops-sandbox-repository`
  1. Set the value for the `ECS_SERVICE` environment variable with `ecs-devops-sandbox-service`
  1. Set the value for the `ECS_CLUSTER` environment variable with `ecs-devops-sandbox-cluster`
  1. Set the value for the `ECS_TASK_DEFINITION` environment variable with `task-definition.json`
  1. Set the value for the `CONTAINER_NAME` environment variable with `ecs-devops-sandbox`

{{% notice warning %}}
Sometimes the starter workflow file changes quicker than these instructions are updated! Be sure to follow the instructions in the starter workflow file for the most up to date steps.
{{% /notice %}}  

Once you've made the changes to the aws.yml workflow file above, select "Start commit". Enter a commit message (or use the default) and select "Commit new file".

## 2. Create IAM user with programmatic access

In the AWS Console, navigate to [IAM](https://console.aws.amazon.com/iam/home) to manage your AWS users.

![](/images/nav-iam.png)

Under "Users", click "Add users" and set the "User name" to `github-actions-user`. For "AWS access type" check "Access key - Programmatic access".

![](/images/iam-create-user.png)

Press "Next: Permissions".

Next, select "Attach existing policies directly" and select the "Create policy" button.

![](/images/iam-create-policy.png)

Navigate to the JSON tab and replace the policy with this one below. Replace the placeholder values (<YOUR_AWS_ACCOUNT_ID> and <YOUR_AWS_REGION>) with your AWS Account ID and the region you are using. You can find your AWS Account ID in the [AWS Console](https://console.aws.amazon.com/), in the upper right drop-down menu under your login info. It is labeled "My Account".

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecs:RegisterTaskDefinition",
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": "arn:aws:ecr:<YOUR_AWS_REGION>:<YOUR_AWS_ACCOUNT_ID>:repository/ecs-devops-sandbox-repository"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ecs:DescribeServices",
                "ecs:UpdateService"
            ],
            "Resource": [
                "arn:aws:ecs:<YOUR_AWS_REGION>:<YOUR_AWS_ACCOUNT_ID>:service/default/ecs-devops-sandbox-service",
                "arn:aws:ecs:<YOUR_AWS_REGION>:<YOUR_AWS_ACCOUNT_ID>:service/ecs-devops-sandbox-cluster/ecs-devops-sandbox-service"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/ecs-devops-sandbox-execution-role"
        }
    ]
}
```

Press "Next: Tags" and then press "Next: Review" and give it a name `ecs-devops-sandbox-policy`. Then press "Create policy". Once the policy is created successfully, close the browser tab to return to the IAM user you were creating.

Press the refresh button to refresh the list of policies and search for the one you just created, `ecs-devops-sandbox-policy`. Check the box for this policy and then press "Next: Tags", "Next: Review", and "Create user" buttons to create the IAM user.

Note down the "AWS ACCESS KEY ID" and the "AWS SECRET ACCESS KEY" to use in the next step. Treat these like a username and password. If you lose the "AWS SECRET ACCESS KEY", you'll need to generate a new one.

## 3. Setup Secrets for GitHub Action

To deploy to ECR we need to give GitHub permission to speak to AWS. To do that we add secrets to the repository.

Secrets are environment variables that are encrypted and can be used by your GitHub action workflow to access external resources like AWS.

1. Go to the Settings tab of your GitHub repository
2. Select Secrets -> Actions
3. Select New repository secret
4. Set Name to `AWS_ACCESS_KEY_ID`
5. Set value to the access key id you copied from the previous section.
6. Select New repository secret to add the second one
7. Set Name to `AWS_SECRET_ACCESS_KEY`
8. Set the value to the secret access key you copied from the previous section.

You should now have two secrets like this:

![](/images/github-actions-secrets.png)

## 4. Update the GitHub action to trigger on push to main

We now need to change the action to trigger the workflow whenever a change is pushed to the main branch.

In your Cloud9 environment terminal, move back into the `demo-github-actions` directory:

```
cd ~/environment/demo-github-actions
```

And pull your changes from GitHub to get the aws.yml workflow file:

```
git pull
```

In your Cloud9 editor, select the gear in the left navigation and select "Show Hidden Files" so that we can see the hidden directories that start with a `.`.

![](/images/cloud9-gear-hidden-files.png)

We want to trigger a build when changes are pushed to the main branch. Ensure the following is in the `.github/workflows/aws.yml` file, with the same indentation:

```
on:
  push:
    branches:
      - main
```

## Add Step to Check Build Commit Status

Now we need to add a step to the workflow to check the build commit status. If the CodeBuild workflow is unsuccessful, the workflow exits.

Add the following as the first step under steps to your `.github/workflows/aws.yml` file:

```
steps:
- name: Check commit status
  id: commit-status
  run: |
    # Check the status of the Git commit
    CURRENT_STATUS=$(curl --url https://api.github.com/repos/${{ github.repository }}/commits/${{ github.sha }}/status --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' | jq -r '.state');
    echo "Current status is: $CURRENT_STATUS"
    while [ "${CURRENT_STATUS^^}" = "PENDING" ];
      do sleep 10;
      CURRENT_STATUS=$(curl --url https://api.github.com/repos/${{ github.repository }}/commits/${{ github.sha }}/status --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' | jq -r '.state');
    done;
    echo "Current status is: $CURRENT_STATUS"
    if [ "${CURRENT_STATUS^^}" = "FAILURE" ];
      then echo "Commit status failed. Canceling execution";
      exit 1;
    fi
... more steps ...    
```

Again, be sure to retain the indentation in this file.

Commit these changes:

```
git add . && git commit -m "Update workflow to run on push to main branch"
```

And push to GitHub:

```
git push origin
```

When this code is pushed to the main branch of your repository on GitHub, CodeBuild is automatically invoked using the webhook and executes the application tests. Let's go check it out next!