+++
title = "Setup Cloud9 Environment"
chapter = false
weight = 2
+++

{{% notice note %}}
Before starting the workshop, ensure that the region selected for the workshop supports ECS, CDK, Cloud9 services. We recommend using us-east-2 (Ohio) or us-west-2 (Oregon) for this workshop.
{{% /notice %}}

We'll use the AWS Cloud 9 service to setup your environment. Cloud9 comes pre-installed with AWS CLI packages, git, python, and many other useful utilities along with built-in IDE functionalities. Once logged in, you can use this Browser based IDE for coding and the bash shell to run terminal commands.

1. In the AWS Console, navigate to the [Cloud9 service](https://console.aws.amazon.com/cloud9/home).

![](/images/nav-cloud9.png)

2. Select "Create environment" button.

![](/images/create-cloud9.png)

Enter a name `Automating CI/CD on AWS with GitHub Actions` and then go through the default selections before pressing "Create environment". Be sure the `t2.micro` instance type is selected to remain in the Free Tier. This development environment will be backed by an [EC2 instance](https://aws.amazon.com/ec2/), which you can think of as a virtual server in the cloud that is secure and resizable and can be scaled up or down depending on your capacity needs.

This will take a few minutes to create the development environment, so you can move on to the next step while you wait.

You'll have IDE capabilities built-in into this environment:

![](/images/cloud9-ide.png)

Next, you'll need to set up your GitHub account.