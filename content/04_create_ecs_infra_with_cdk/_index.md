+++
title = "2. Create ECS Infrastructure"
chapter = true
weight = 40
+++

# Create ECS Infrastructure

Next, we'll create the Amazon ECS Infrastructure we'll deploy our Flask app to using the Cloud Development Kit (CDK). Go back to your Cloud9 development environment, where we'll complete the steps on this page.

## 1. Update the aws-cdk command line package

First we'll update the aws-cdk command line tools so that we're using the latest and greatest. In your Cloud9 environment terminal, run the following command:

```
npm install -g aws-cdk
```

{{% notice note %}}
If you encounter an error that a "file already exists", you can force the install by adding the force flag: `npm install -g aws-cdk --force`
{{% /notice %}}

## 2. Bootstrap the CDK environment

You'll need to bootstrap each environment you deploy to. Bootstrapping will ensure that the AWS resources required by your deployment are provisioned and available. To do this, find your account number and region and execute this command:

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

## 3. Initialize the ECS sandbox project
Change directory to one level above your Flask project directory:

```
cd ..
```

From the command line, make the `ecs-devops-sandbox-cdk` project directory and move into it:

```
mkdir ecs-devops-sandbox-cdk && cd ecs-devops-sandbox-cdk
```

## 4. Initialize a CDK project:

```
cdk init --language python 
```

## 5. Install project requirements

```
source .venv/bin/activate
python -m pip install -r requirements.txt
```

## 6. Update CDK code

Replace the contents of the file `ecs_devops_sandbox_cdk/ecs_devops_sandbox_cdk_stack.py` (automatically created by the CDK) with the code below. Be sure to retain the same indentation as below and save the changes.

```
import aws_cdk as cdk
import aws_cdk.aws_ecr as ecr
import aws_cdk.aws_ec2 as ec2
import aws_cdk.aws_ecs as ecs
import aws_cdk.aws_iam as iam
# import aws_cdk.aws_ecs_patterns as ecs_patterns

class EcsDevopsSandboxCdkStack(cdk.Stack):

    def __init__(self, scope: cdk.App, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

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

        #
        # Option 1: Creates service, container, and task definition without creating a load balancer
        #   and other costly resources. Containers will not be publicly accessible.
        #
        task_definition = ecs.FargateTaskDefinition(self,
            "ecs-devops-sandbox-task-definition",
            execution_role=execution_role,
            family="ecs-devops-sandbox-task-definition")

        container = task_definition.add_container(
            "ecs-devops-sandbox",
            image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample"),
            logging=ecs.LogDrivers.aws_logs(stream_prefix="ecs-devops-sandbox-container")
        )

        service = ecs.FargateService(self,
            "ecs-devops-sandbox-service",
            cluster=cluster,
            task_definition=task_definition,
            service_name="ecs-devops-sandbox-service")

        # END Option 1

        #
        # Option 2: Creates a load balancer and related AWS resources using the ApplicationLoadBalancedFargateService construct.
        #   These resources have non-trivial costs if left provisioned in your AWS account, even if you don't use them. Be sure to
        #   clean up (cdk destroy) after working through this exercise.
        #
        # Comment out option 1 and uncomment the code below. Uncomment the aws_cdk.aws_ecs_patterns import at top of file.
        #
        # task_image_options = ecs_patterns.ApplicationLoadBalancedTaskImageOptions(
        #     family="ecs-devops-sandbox-task-definition",
        #     execution_role=execution_role,
        #     image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample"),
        #     container_name="ecs-devops-sandbox",
        #     container_port=8080,
        #     log_driver=ecs.LogDrivers.aws_logs(stream_prefix="ecs-devops-sandbox-container")
        # )
        #
        # ecs_patterns.ApplicationLoadBalancedFargateService(self, "ecs-devops-sandbox-service",
        #     cluster=cluster,
        #     service_name="ecs-devops-sandbox-service",
        #     desired_count=2,
        #     task_image_options=task_image_options,
        #     public_load_balancer=True
        # )
        #
        # END Option 2
```

{{% notice note %}}
This sample code contains two options to create the required infrastructure.

Option 1 Creates service, container, and task definition without creating a load balancer and other costly resources. Containers will not be publicly accessible. This is the default and recommended approach for this workshop.

Option 2 Creates a load balancer and related AWS resources using the ApplicationLoadBalancedFargateService construct, allowing the container to be publicly accessible. These resources have non-trivial costs if left provisioned in your AWS account, even if you don't use them. Be sure to clean up (cdk destroy) after working through this exercise.
{{% /notice %}}

## 7. Deploy to create the infrastructure

Let's deploy this to create the AWS infrastructure with the CDK code we wrote:

```
cdk deploy
```

Confirm the changes with `y` and press enter. This will take about five minutes to complete. While you're waiting, you can move to the next step.
