---
link: https://thenewstack.io/git-push-how-to-use-the-cli-to-interact-with-github/
byline: Jack Wallen
site: The New Stack
date: 2024-09-14T16:00
excerpt: By using git with GitHub, you are able to collaborate on code with anyone else able to access the repository. Here's how.
twitter: https://twitter.com/@jlwallen
slurped: 2025-03-14T15:47
title: "Git Push: How to Use the CLI to Interact with GitHub"
tags:
  - git
  - token
  - push
  - cli
  - GitHub-Account
---

So you’ve finally added the [git command line interface](https://cli.github.com/) to your developer workflow but you’re not sure how to make it work with your team’s [GitHub repository](https://thenewstack.io/dont-mess-with-the-master-working-with-branches-in-git-and-github/). By using git with GitHub, you can then collaborate on code with anyone else able to access the repository.

This is a crucial feature of GitHub (and why it exists in the first place). Collaboration is at the heart of such tools and getting the git CLI and GitHub to communicate with one another is essential, especially if you have any intention of working with a team.

But getting the CLI to communicate with your team’s GitHub repository isn’t exactly the most intuitive task you’ll undertake. In fact, it can be quite confusing, which can quickly become a very frustrating issue if you’re uncertain how to make it work.

That’s where I come in. I want to walk you through the process of connecting your local Git command line with a GitHub repository. Once you see how it works, you shouldn’t have any problems in the future.

Let’s make that happen.

## What You’ll Need

The requirements for this are important, so pay close attention. Here’s what you’ll need to connect git with GitHub:

- [Git installed](https://thenewstack.io/take-your-first-steps-with-git/) on your local machine.
- A valid GitHub account.
- A personal access token for your GitHub account with the correct permissions.
- A valid GitHub repository.

Without all of those things above, this will not work. Let me address each point before we continue.

First, you must have Git installed on your local machine. This can be done on Linux, macOS  or Windows. Next, you must have a valid GitHub account, which can be free or paid. You’ll also need a valid personal access token because you can no longer work with standard username/password credentials.

To create a personal access token, do the following:

- Log into your GitHub account.
- Click your profile icon and then click Settings.
- In the left navigation, click Developer Settings.
- Click Personal Access Tokens in the left navigation and then click Fine-grained tokens.
- Click Generate New Token.
- Fill out the necessary information.
- Make sure to select All Repositories (under Repository Access).
- Under Repository Permissions, make sure to go through each permission and select the proper permissions from the drop-down. For example, you’ll want to make sure things like Actions have read and write access.
- When finished, click Generate token.

Once you’ve taken care of the above, make sure to copy the token and save it in a secure location.

Note: If your team’s GitHub page doesn’t allow you to create the new token, you’ll need to request that your admin take care of it.

Finally, you’ll need a valid GitHub repository to connect to. From that repository, you want to make sure you copy the URL for the repository by clicking the green Code drop-down and then clicking the Copy icon for the address (Figure 1).

![Test repository](https://cdn.thenewstack.io/media/2024/09/a9210780-git1.jpg)

Figure 1: Don’t bother using this repository as it’s only used for testing purposes.

Once you have the link, it’s time to make the git CLI work.

## Using the git CLI with GitHub

The first thing we need to do is set our global username and email address for Git. This is done with the following two commands:  

|   |   |
|---|---|
||git config --global user.name "NAME"|

|   |   |
|---|---|
||git config --global user.email "EMAIL"|

  
Where `NAME` is your real name and `EMAIL` is the email address associated with your GitHub account.

Fun times.

Next, create a directory to be used on your local machine. We’ll call this repository `new-project` and create the directory with:  

  
Change into that directory with:  

  
Initialize the repository with:  

  
We can now connect the local repository with the GitHub repository with the command:  

|   |   |
|---|---|
||git remote add origin URL|

  
Where URL is the link you copied from the GitHub repository.

Next, pull the content of the remote repository with:  

  
Note: You might have to change `master` to the branch name you need to work with.

You should now see all of the code from the project in your local directory.

Let’s say the first thing you need to do is edit (or create) the README file. Open the file with your favorite editor or integrated development environment and make some changes to the file. Once you’ve done that, add the file to the staging area with:  

  
Next, make your first commit with:  

|   |   |
|---|---|
||git commit -m "Added new information to README file"|

  
Issue a push to send the changes to the remote repository with:  

  
You’ll be prompted for your GitHub username and password. The password is actually the personal access token you created earlier.

Upon successful authentication, you’ll see the changes were pushed to the remote repository. You can always go back to GitHub and view the file to verify the changes were made.

You can add files, edit code, and do whatever it is you need for the project, as long as you make sure to follow the usual Git workflow, which you can read about in my post, [“Need To Know Git? Start Here.”](https://thenewstack.io/need-to-know-git-start-here/)

When next we visit Git, we’ll create a local repository that can be accessed by other developers on your LAN.