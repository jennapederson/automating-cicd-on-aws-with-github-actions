+++
title = "6. Fix the broken build (optional)"
chapter = true
weight = 80
+++

# Fix the broken build

With the AWS CodeBuild project and GitHub workflow in place, we can add new features to our application that automatically deploy to Amazon ECS if all tests pass. If our application is broken, it does not deploy. But how do we fix a broken build?

Open the app.py file and change this line:

```
return "".join(reversed(random_string))
```

to

```
return "Broken unit test!"
```

This change to the application code (app.py) will break the unit test.

Commit this change:

```
git add . && git commit -m "Break the unit test"
```

And push to GitHub:

```
git push origin
```

The GitHub workflow detects the status of the commit to be a failure success, and the remaining steps in the workflow successfully execute the deployment of the application to ECS.

Because our GitHub workflow detected the status of the commit to be a failure, the workflow did not proceed past the “Check commit status” step.

TBD IMAGE - failed job on GitHub

If we revert our change and push the fix to GitHub again, we can then see that it's now successful.

TBD - successful github job status

Finally, we can see that the ECS Task Definition has been updated to reflect the new application container image. Note that the Git commit hash is used as the container image tag.