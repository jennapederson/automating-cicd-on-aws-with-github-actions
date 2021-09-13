+++
title = "Setup Cloud9 Environment"
chapter = false
weight = 2
+++

{{% notice note %}}
Before starting the workshop, ensure that the region selected for the workshop supports ECS, CDK, Cloud9 services. Recommend using us-east-2 (Ohio) or us-west-2 (Oregon) for the Workshop.
{{% /notice %}}

We'll use the AWS Cloud 9 service to setup your environment. Cloud9 comes pre-installed with AWS CLI packages, git, python, and many other useful utilities along with built-in IDE functionalities. Once logged in, you can use this Browser based IDE for coding and the bash shell to run terminal commands.

1. In the AWS Console, navigate to the Cloud9 service.

![](/images/nav-cloud9.png)

2. Select Create environment button.

![](/images/create-cloud9.png)

Go through the default selections and wait for the environment to be created. Be sure the `t2.micro` instance type is selected to remain in the Free Tier. This will launch an EC2 instance with your environment name and once fully up, you will get access to your development environment.

You have IDE capabilities built-in into this environment:

![](/images/cloud9-ide.png)

Now you're ready for the workshop!