+++
title = "4. Create GitHub Action"
chapter = true
weight = 60
+++

# Create a GitHub Action

AWS provides a starter GitHub workflow that takes advantage of the AWS open-source GitHub Actions to build and deploy containers on ECS for each commit to the main branch of a repository.

## 1. Setup GitHub Action

Go to your `demo-github-actions` repository on GitHub. On the Actions tab, find "Deploy to Amazon ECS" and select "Set up this workflow".

![](/images/gh-action-choose-workflow-1.png)

![](/images/gh-action-choose-workflow-2.png)

Following the instructions outlined in the starter workflow file for using this workflow with our application, we will reference the ECR repository, ECS task definition, ECS cluster, and ECS service that we created previously. Here are the specific steps to complete if you don't want to read through the starter workflow instructions:

  1. Replace the `ECR_REPOSITORY` environment variable with  `ecs-devops-sandbox-repository`
  1. Set the value of aws-region to `us-east-1` (or whatever region is shown in the upper right of the AWS Console)
  1. Set the value of container-name to `ecs-devops-sandbox`
  1. Set the value of service to `ecs-devops-sandbox-service`

Once you've made the changes to the aws.yml workflow file above, select "Start commit". Enter a commit message (or use the default) and select "Commit new file".

## 2. Create IAM user with programmatic access

In the AWS Console, navigate to [IAM](https://console.aws.amazon.com/iam/home) to manage your AWS users.

![](/images/nav-iam.png)

Under "Users", click "Add users" and set the "User name" to `github-actions-user`. For "AWS access type" check "Access key - Programmatic access".

![](/images/iam-create-user.png)

Press "Next: Permissions".

Next, select "Attach existing policies directly" and check "AdministratorAccess".

![](/images/iam-policy.png)

Press "Next: Tags" and then press "Next: Review" and "Create user".

Note down the "AWS ACCESS KEY ID" and the "AWS SECRET ACCESS KEY" to use in the next step. Treat these like a username and password. If you lose the "AWS SECRET ACCESS KEY", you'll need to generate a new one.

## 3. Setup Secrets for GitHub Action

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

Instead of triggering the workflow when a release is created, we want to change it to trigger when changes are pushed to the main branch.

In the `.github/workflows/aws.yml` file, replace these lines:

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

Be sure to retain the indentation you see above as it's important in this file.

## Add Step to Check Build Commit Status

Now we need to add a step to workflow to check the build commit status. If the CodeBuild workflow is unsuccessful, the workflow exists.

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

{{%expand "View the full `.github/workflows/aws.yml` file" %}}

```
# This workflow will build and push a new container image to Amazon ECR,
# and then will deploy a new task definition to Amazon ECS, when a release is created
#
# To use this workflow, you will need to complete the following set-up steps:
#
# 1. Create an ECR repository to store your images.
#    For example: `aws ecr create-repository --repository-name my-ecr-repo --region us-east-2`.
#    Replace the value of `ECR_REPOSITORY` in the workflow below with your repository's name.
#    Replace the value of `aws-region` in the workflow below with your repository's region.
#
# 2. Create an ECS task definition, an ECS cluster, and an ECS service.
#    For example, follow the Getting Started guide on the ECS console:
#      https://us-east-2.console.aws.amazon.com/ecs/home?region=us-east-2#/firstRun
#    Replace the values for `service` and `cluster` in the workflow below with your service and cluster names.
#
# 3. Store your ECS task definition as a JSON file in your repository.
#    The format should follow the output of `aws ecs register-task-definition --generate-cli-skeleton`.
#    Replace the value of `task-definition` in the workflow below with your JSON file's name.
#    Replace the value of `container-name` in the workflow below with the name of the container
#    in the `containerDefinitions` section of the task definition.
#
# 4. Store an IAM user access key in GitHub Actions secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
#    See the documentation for each action used below for the recommended IAM policies for this IAM user,
#    and best practices on handling the access key credentials.

on:
  push:
    branches:
      - main

name: Deploy to Amazon ECS

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    environment: production

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

    - name: Checkout
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ecs-devops-sandbox-repository
        IMAGE_TAG: ${{ github.sha }}
      run: |
        # Build a docker container and
        # push it to ECR so that it can
        # be deployed to ECS.
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"

    - name: Fill in the new image ID in the Amazon ECS task definition
      id: task-def
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: task-definition.json
        container-name: ecs-devops-sandbox
        image: ${{ steps.build-image.outputs.image }}

    - name: Deploy Amazon ECS task definition
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ steps.task-def.outputs.task-definition }}
        service: ecs-devops-sandbox-service
        cluster: ecs-devops-sandbox-cluster
        wait-for-service-stability: true

```

{{% /expand%}}

Commit these changes:

```
git add . && git commit -m "Update workflow to run on push to main branch"
```

And push to GitHub:

```
git push origin
```

When this code is pushed to the main branch of your repository on GitHub, CodeBuild is automatically invoked using the webhook and executes the application tests. Let's go check it out next!