---
title: "Automating CI/CD on AWS with GitHub Actions"
chapter: true
weight: 1
---
# Automating CI/CD on AWS with GitHub Actions

## What We'll Build

In this workshop, we'll show organizations using GitHub as a source code repository can use GitHub Actions to build a complete CI/CD pipeline for applications deployed on Amazon ECS. This workshop also illustrates how AWS CodeBuild can be used with GitHub Actions to execute application tests as part of a complete CI/CD pipeline.

#### The workshop is divided into seven sections. You will:

- Create a basic Flask web app
- Create Amazon ECS Infrastructure using the CDK
- Create an AWS CodeBuild Project
- Create a GitHub Action
- Check the build results
- Fix the broken build
- Clean up your AWS resources

## The Architecture

Below is the architecture for this workshop.

![](images/architecture.png)

