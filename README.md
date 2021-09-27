# Automating CI/CD on AWS with GitHub Actions

## Build the static site with Hugo

After cloning the repo, make sure to clone the submodule:

```sh
cd automating-cicd-on-aws-with-github-actions
git submodule init
git submodule update --checkout --recursive
```

### Run Hugo locally:

`hugo serve`

Point your browser to http://localhost:1313/.

## Instructor Clean Up Instructions

In addition to the Clean Up step in the workshop, if you are running through this multiple times for a workshop, be sure to delete these resources as well:

IAM role: codebuild-ecs-devops-sandbox-service-role
IAM policy: CodeBuildBasePolicy-ecs-devops-sandbox-us-east-1
