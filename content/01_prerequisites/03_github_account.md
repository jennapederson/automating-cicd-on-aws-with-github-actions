+++
title = "GitHub Account"
chapter = false
weight = 3
+++

You'll need an active GitHub account where you can create a repository. If you don't already have one, create a free individual account by following the prompts [here](https://github.com/signup).

Then, you'll need to a way to authenticate with GitHub from the command line in your Cloud9 environment. Setup a [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) or an [SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) to push local changes from the Cloud9 development environment to your GitHub repository.

{{% notice warning %}}
In the interest of time, we recommend creating a [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token). The rest of this workshop assumes you're working with a token which you'll use when prompted for a password at the command line.
{{% /notice %}}

{{% notice note %}}
Create your personal access token with "repo" and "workflow" scopes. Then save the personal access token like you would a password. You'll need it later.
{{% /notice %}}

Now that you have a GitHub account to use, you're ready for the workshop!