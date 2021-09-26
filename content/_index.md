---
title: "Automating CI/CD on AWS with GitHub Actions"
chapter: true
weight: 1
---
# Automating CI/CD on AWS with GitHub Actions

## What We'll Build

In this workshop, we'll how show organizations using GitHub as a source code repository can use GitHub Actions to build a complete CI/CD pipeline for applications deployed on Amazon ECS. This workshop also illustrates how AWS CodeBuild can be used with GitHub Actions to execute application tests as part of a complete CI/CD pipeline.

#### The workshop is divided into seven sections. You will:

1. Create a basic Flask web app
1. Create an Amazon ECS Infrastructure using the CDK
1. Create an AWS CodeBuild Project
1. Create a GitHub Action
1. Check the build results
1. Fix the broken build (optional)
1. Clean up your AWS resources

## The Architecture

Below is the architecture for this workshop.

![](images/architecture.png)

