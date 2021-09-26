+++
title = "2. Create ECS Infrastructure"
chapter = true
weight = 40
+++

# Create ECS Infrastructure

Next, we'll create the Amazon ECS Infrastructure we'll deploy our Flask app to using the Cloud Development Kit (CDK).

## 1. Update the aws-cdk command line package

First we'll update the aws-cdk command line tools so that we're using the latest and greatest. In your Cloud9 environment terminal, run the following command:

```
npm install -g aws-cdk
```

{{% notice note %}}
If you encounter an error that a "file already exists", you can force the install by adding the force flag: `npm install -g aws-cdk --force`

{{% /notice %}}

## 2. Initialize the ECS sandbox project
Change directory to one level above your Flask project directory:

```
cd ..
```

From the command line, make the `ecs-devops-sandbox-cdk` project directory and move into it:

```
mkdir ecs-devops-sandbox-cdk && cd ecs-devops-sandbox-cdk
```

## 3. Initialize a CDK project:

```
cdk init --language python 
```

## 4. Install project requirements

```
pip install -r requirements.txt
```

## 5. Install other requirements

```
pip install aws_cdk.aws_ec2 aws_cdk.aws_ecs aws_cdk.aws_ecr aws_cdk.aws_iam
```

## 6. Update CDK code

Replace the contents of the file `ecs_devops_sandbox_cdk/ecs_devops_sandbox_cdk_stack.py` (automatically created by the CDK) with the code below. Be sure to retain the same indentation as below and save the changes.

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

## 7. Deploy to create the infrastructure

Let's deploy this to create the AWS infrastructure with the CDK code we wrote:

```
cdk deploy
```

Confirm the changes with `y` and press enter. This will take about five minutes to complete. While you're waiting, you can move to the next step.