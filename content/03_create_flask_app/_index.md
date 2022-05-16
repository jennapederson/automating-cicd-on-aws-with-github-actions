+++
title = "1. Create a basic Flask web app"
chapter = true
weight = 30
+++

# Create a basic Flask web app

## 1. Initialize project

To get started creating the components necessary to build our example architecture, we need a project directory and GitHub repository.

Go to your Cloud9 development environment, and find the command line tab in the lower right.

![](/images/cloud9-command-line.png)

From the command line, make the `demo-github-actions` project directory and move into it using this command:

```
mkdir demo-github-actions && cd demo-github-actions
```

Then, let's update the default branch in git to use `main`:

```
git config --global init.defaultBranch main
```

And initialize the git repository:

``` 
git init
```

Create the following files:

{{% notice tip %}}
You can create the empty files quickly using the `touch` command. If you'd like to do it manually, you can use the File -> New File menu to create each of the files below.
{{% /notice %}}

```
touch Dockerfile app.py app_test.py buildspec.yml requirements.txt task-definition.json
```

## 2. Create a basic Flask web app

We are going to write a very basic web app using Flask. Flask is a micro web framework written in Python.

Open the `app.py` file from the left navigation pane:

![](/images/cloud9-open-file.png)

And add this basic Flask code to `app.py` and save the file.

```
"""Main application file"""
from flask import Flask
app = Flask(__name__)

@app.route("/")
def health_check():
    return "Healthy!"

@app.route('/<random_string>')
def returnBackwardsString(random_string):
    """Reverse and return the provided URI"""
    return "".join(reversed(random_string))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

This creates the Flask app, adds a route to reverse any string that is provided, and then runs the app and listens on port 8080.

## 3. Update the test file

The `app_test.py` file is a unit test file for `app.py` to ensure that the string is reversed correctly.

In the `app_test.py` file, add:

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

Save the file. 

## 4. Update the requirements.txt file

The `requirements.txt` file contains dependencies for `app.py`. In this file, add the flask requirement:

```
flask>=2.1.2
```

Save the file. 

## 5. Update the buildspec.yml file

The `buildspec.yml` file contains instructions for AWS CodeBuild to run our unit tests. Refer to the [Build Specification Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) for more information about the capabilities and syntax available for buildspec files.

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

Save the file. 

## 6. Update the Dockerfile

We are creating a container-based application, so we are going to need a `Dockerfile`. A `Dockerfile` gives the instructions to [Docker](https://www.docker.com/) on how to create a container from the app.

```
FROM python:3
# Set application working directory
WORKDIR /usr/src/app
# Install requirements
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
# Install application
COPY app.py ./
# Open port app runs on
EXPOSE 8080
# Run application
CMD python app.py
```

Save the file. 

## 7. Update the task-definition.json file

The `task-definition.json` file contains specifications for an ECS Task Definition. In this file, replace the placeholder value (<YOUR_AWS_ACCOUNT_ID>) with your AWS Account ID. You can find your AWS Account ID in the [AWS Console](https://console.aws.amazon.com/), in the upper right drop-down menu under your login info. It is labeled "My Account".

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

Be sure all of your files have been saved in the editor.

## 8. Commit your changes

The end result of this step should be a git repository containing all application, test, and deployment files. Commit your changes:

```
git add . && git commit -m "Initial commit - basic Flask app"
```

## 9. Create GitHub repository

Now we need to create a repository on GitHub and push our local changes to it. Go to https://github.com and select the "New" button to create a new repository. Enter `demo-github-actions` for the repository name and set it to "Private". Select the "Create repository" button to create your new repository.

{{% notice note %}}
You can create a public repository, however since we are putting our AWS account ID in the `task-definition.json` file, a private repository will keep it more secure.
{{% /notice %}}

![](/images/create-github-repo.png)

## 10. Push local changes to GitHub

Go back to the terminal in your Cloud9 development environment and add the GitHub remote repository as origin:

```
git remote add origin https://github.com/<YOUR GITHUB PROFILE>/demo-github-actions.git
```

You can also find this command at the bottom of your brand new repository on GitHub. Just be sure to select HTTPS if you're using a personal token.

Then push your local changes:

```
git push -u origin main
```

When prompted for a username and password, enter your GitHub username and the personal access token you created earlier. If everything goes well, your code will show up in the repository you created on GitHub.

Now, we'll create the Amazon ECS Infrastructure that we'll be deploying our app to using the Cloud Development Kit (CDK).
