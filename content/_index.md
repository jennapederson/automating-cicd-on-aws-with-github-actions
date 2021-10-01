---
title: "Automating CI/CD on AWS with GitHub Actions"
chapter: true
weight: 1
---
# Automating CI/CD on AWS with GitHub Actions

## What We'll Build

In this workshop, we'll show how organizations using GitHub as a source code repository can use GitHub Actions to build a complete CI/CD pipeline for applications deployed on Amazon ECS. This workshop also illustrates how AWS CodeBuild can be used with GitHub Actions to execute application tests as part of a complete CI/CD pipeline.

#### The workshop is divided into seven sections. You will:

1. Create a basic Flask web app
1. Create an Amazon ECS Infrastructure using the CDK
1. Create an AWS CodeBuild Project
1. Create a GitHub Action
1. Check the build results
1. Fix the broken build (optional)
1. Clean up your AWS resources

## The Architecture

This architecture represents a complete CI/CD pipeline that uses a GitHub workflow to automatically coordinate building, testing, and deploying an application to ECS for every push to the repository.  This GitHub workflow uses the AWS open-source GitHub Actions to coordinate build and deploy tasks, and uses a service called AWS CodeBuild to execute application tests. We also introduce a custom GitHub Action to this pipeline to evaluate the status of CodeBuild tests.

We'll create a git repository with a simple Python web application and accompanying infrastructure files. We’re going to first build our web app using Flask and Python. Then we’re going to create the ECS Fargate Infrastructure to run our containers and save the container image into the Registry using the Cloud Development Kit, or CDK. The third step will be setting up the AWS Code Build Project, and the last step will be to create the Github action and tie it all together.

Once it’s all hooked up, it’s just like a Rube Goldberg machine. Whenever you push code to the Github repository, that’ll kick it off. The CI/CD pipeline will execute unit tests, build a container image, upload the container image to ECR, and update the ECS Task Definition to deploy our new code into production.

![](images/architecture.png)

