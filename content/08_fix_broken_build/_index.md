+++
title = "6. Fix the broken build (optional)"
chapter = true
weight = 80
+++

# Fix the broken build

With the AWS CodeBuild project and GitHub workflow in place, we can add new features to our application that automatically deploy to Amazon ECS if all tests are passed.Right now, our application does not deploy because the tests are broken. Let's fix it!

Open the app.py file and change this line:

```
return "Broken unit test!"
```

to

```
return "".join(reversed(random_string))
```

This change to the application code (app.py) will fix the failing unit test.

Commit and push this change to origin:

```
git add .
git commit -m "Break the unit test"
git push origin
```

The GitHub workflow detects the status of the commit to be success, and the remaining steps in the workflow successfully execute the deployment of the application to ECS.

TBD - github job status

Finally, we can see that the ECS Task Definition has been updated to reflect the new application container image.  Note that the Git commit hash is used as the container image tag.