# Create basic Flask web app

## Create GitHub repo and initialize project

To get started creating the components necessary to build our example architecture, we need a project directory and GitHub repository.

From the command line, make the `demo-github-actions` project directory and move into it:

```
mkdir demo-github-actions && cd demo-github-actions
```

Initialize the git repository:

```
git init
```

Create the following files:

```
touch Dockerfile app.py app_test.py buildspec.yml requirements.txt task-definition.json
```

Open VSCode

```
code .
```

## Create a basic Flask web app

We are going to write a very basic web app using Flask. Flask is a micro web framework written in Python. Add this basic Flask code to the app.py file. 

```
"""Main application file"""
from flask import Flask
app = Flask(__name__)
```

In the same file, create a flask route which will reverse any string that is provided.

```
@app.route('/<random_string>')
def returnBackwardsString(random_string):
    """Reverse and return the provided URI"""
    return "Broken unit test!"
```

Add some code to run the app and listen on port 8080

```
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```
## Create the test file

The app_test.py file is a unit test file for app.py to ensure that the string is reversed correctly.

In the app_test.py file, add:

```
"""Unit test file for app.py"""
from app import returnBackwardsString
import unittest

class TestApp(unittest.TestCase):
    """Unit tests defined for app.py"""

    def test_return_backwards_string(self):
        """Test return backwards simple string"""
        random_string = "This is my test string"
        random_string_reversed = "gnirts tset ym si sihT"
        self.assertEqual(random_string_reversed, returnBackwardsString(random_string))

if __name__ == "__main__":
    unittest.main()
```

## Create the requirements.txt file

The requirements.txt file contains dependencies for app.py. In this file, add the flask requirement:

```
flask==2.0.1
```

## Create the buildspec.yml file


The buildspec.yml file contains instructions for AWS CodeBuild to run our unit tests. Refer to the [Build Specification Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) for more information about the capabilities and syntax available for buildspec files.

```
version: 0.2
phases:
  install:
    runtime-versions:
      python: 3.8
  pre_build:
    commands:
      - pip install -r requirements.txt
      - python app_test.py
```

## Create a Docker File name Dockerfile

We are creating a container-based application, so we are going to need a Dockerfile. A Dockerfile gives the instructions to [Docker](https://www.docker.com/) on how to create a container from the app.

```
FROM python:3
# Set application working directory
WORKDIR /usr/src/app
# Install requirements
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
# Install application
COPY app.py ./
# Run application
CMD python app.py
```

## Create a task-definition.json file

The task-definition.json file contains specifications for an ECS Task Definition. In this file, replace the placeholder value (<YOUR_AWS_ACCOUNT_ID>) with your AWS Account ID. You can find your AWS Account ID in the AWS Console, in the upper right drop-down menu under your login info. It is labeled "My Account".

```
{
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "inferenceAccelerators": [],
    "containerDefinitions": [
        {
            "name": "ecs-devops-sandbox",
            "image": "ecs-devops-sandbox-repository:00000",
            "resourceRequirements": null,
            "essential": true,
            "portMappings": [
                {
                    "containerPort": "8080",
                    "protocol": "tcp"
                }             
            ]
        }
    ],
    "volumes": [],
    "networkMode": "awsvpc",
    "memory": "512",
    "cpu": "256",
    "executionRoleArn": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/ecs-devops-sandbox-execution-role",
    "family": "ecs-devops-sandbox-task-definition",
    "taskRoleArn": "",
    "placementConstraints": []
}
```

The end result of this step should be a GitHub repository containing all application, test, and deployment files.

## Commit your changes and push your code to GitHub origin.

Commit your changes:

```
git add . && git commit -m "Initial commit - basic Flask app"
```

# Create Amazon ECS Infrastructure using the Cloud Development Kit (CDK)


## Initialize the ECS sandbox project
Change directory to one level above your Flask project directory:

```
cd ..
```

From the command line, make the `ecs-devops-sandbox-cdk` project directory and move into it:

```
mkdir ecs-devops-sandbox-cdk && cd ecs-devops-sandbox-cdk
```

### Initialize a CDK project:

```
cdk init --language python 
```

### Install requirements

```
pip install -r requirements.txt
```

### Install requirements

```
pip install aws_cdk.aws_ec2 aws_cdk.aws_ecs aws_cdk.aws_ecr aws_cdk.aws_iam
```

## Update CDK code
Replace the contents of the file ecs_devops_sandbox_cdk/ecs_devops_sandbox_cdk_stack.py (automatically created by the CDK) with the code below. Be sure to retain the same indentation as below.

```
"""AWS CDK module to create ECS infrastructure"""
from aws_cdk import (core, aws_ecs as ecs, aws_ecr as ecr, aws_ec2 as ec2, aws_iam as iam)

class EcsDevopsSandboxCdkStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

       
```

### Create the ECR Repository

In the same file, add:

```
        ecr_repository = ecr.Repository(self,
                                        "ecs-devops-sandbox-repository",
                                        repository_name="ecs-devops-sandbox-repository")
```

Add the following code to create the ECS Cluster (and VPC)

```
        vpc = ec2.Vpc(self,
                      "ecs-devops-sandbox-vpc",
                      max_azs=3)
        cluster = ecs.Cluster(self,
                              "ecs-devops-sandbox-cluster",
                              cluster_name="ecs-devops-sandbox-cluster",
                              vpc=vpc)
```

Create the ECS Task Definition with placeholder container (and named Task Execution IAM Role)

```
        execution_role = iam.Role(self,
                                  "ecs-devops-sandbox-execution-role",
                                  assumed_by=iam.ServicePrincipal("ecs-tasks.amazonaws.com"),
                                  role_name="ecs-devops-sandbox-execution-role")
        execution_role.add_to_policy(iam.PolicyStatement(
            effect=iam.Effect.ALLOW,
            resources=["*"],
            actions=[
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
                ]
        ))
        task_definition = ecs.FargateTaskDefinition(self,
                                                    "ecs-devops-sandbox-task-definition",
                                                    execution_role=execution_role,
                                                    family="ecs-devops-sandbox-task-definition")
        container = task_definition.add_container(
            "ecs-devops-sandbox",
            image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample")
        )
```

Create the ECS Service

```
        service = ecs.FargateService(self,
                                     "ecs-devops-sandbox-service",
                                     cluster=cluster,
                                     task_definition=task_definition,
                                     service_name="ecs-devops-sandbox-service")
```


{{%expand "View the complete ecs_devops_sandbox_cdk/ecs_devops_sandbox_cdk_stack.py file" %}}

```
"""AWS CDK module to create ECS infrastructure"""
from aws_cdk import (core, aws_ecs as ecs, aws_ecr as ecr, aws_ec2 as ec2, aws_iam as iam)

class EcsDevopsSandboxCdkStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        ecr_repository = ecr.Repository(self,
                                            "ecs-devops-sandbox-repository",
                                            repository_name="ecs-devops-sandbox-repository")

        vpc = ec2.Vpc(self,
                        "ecs-devops-sandbox-vpc",
                        max_azs=3)
        cluster = ecs.Cluster(self,
                                "ecs-devops-sandbox-cluster",
                                cluster_name="ecs-devops-sandbox-cluster",
                                vpc=vpc)

        execution_role = iam.Role(self,
                                    "ecs-devops-sandbox-execution-role",
                                    assumed_by=iam.ServicePrincipal("ecs-tasks.amazonaws.com"),
                                    role_name="ecs-devops-sandbox-execution-role")
        execution_role.add_to_policy(iam.PolicyStatement(
            effect=iam.Effect.ALLOW,
            resources=["*"],
            actions=[
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
                ]
        ))
        task_definition = ecs.FargateTaskDefinition(self,
                                                    "ecs-devops-sandbox-task-definition",
                                                    execution_role=execution_role,
                                                    family="ecs-devops-sandbox-task-definition")
        container = task_definition.add_container(
            "ecs-devops-sandbox",
            image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample")
        )

        service = ecs.FargateService(self,
                                            "ecs-devops-sandbox-service",
                                            cluster=cluster,
                                            task_definition=task_definition,
                                            service_name="ecs-devops-sandbox-service")
```

{{% /expand%}}

## Deploy to create the infrastructure


Let's deploy this to create the AWS infastructure with the CDK code we wrote:

```
cdk deploy
```

Confirm with `y` the changes. This will take about 5 minutes to complete.

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

# Check the build results

When this code is pushed to the main branch of the repository, we can see that CodeBuild is automatically invoked via webhook and executes the application tests. And it looks like our first test failed!

TBD IMAGE - CodeBuild

CodeBuild then reports the unsuccessful test execution to GitHub.

TBD IMAGE - GitHub failure

Because our GitHub workflow detected the status of the commit to be a failure, the workflow did not proceed past the “Check commit status” step.

TBD IMAGE - failed job on GitHub

# Fix the broken build

With the AWS CodeBuild project and GitHub workflow in place, we can add new features to our application that automatically deploy to Amazon ECS if all tests are passed.Right now, our application does not deploy because the tests are broken. Let's fix it!

Open the app.py file and change this line:

```
return "Broken unit test!"
```

to

```
return "".join(reversed(random_string))
```

This change to the application code (app.py) will fix the failing unit test.

Commit and push this change to origin:

```
git add .
git commit -m "Break the unit test"
git push origin
```

The GitHub workflow detects the status of the commit to be success, and the remaining steps in the workflow successfully execute the deployment of the application to ECS.

TBD - github job status

Finally, we can see that the ECS Task Definition has been updated to reflect the new application container image.  Note that the Git commit hash is used as the container image tag.

