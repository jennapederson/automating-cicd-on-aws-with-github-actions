+++
title = "Create a basic Flask web app"
chapter = true
weight = 30
+++

# Create a basic Flask web app

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