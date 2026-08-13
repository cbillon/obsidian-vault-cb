---
link: https://thenewstack.io/git-set-up-a-local-repository-accessible-by-lan/
byline: Jack Wallen
site: The New Stack
date: 2024-09-28T18:00
excerpt: How to quickly and securely deploy a Git repository on your Local Area Network for you and other team members to use.
twitter: https://twitter.com/@jlwallen
slurped: 2026-06-27T19:18
title: "Git: Set Up a Local Repository Accessible by LAN"
tags:
  - git
---

A [Git repository](https://thenewstack.io/need-to-know-git-start-here/) simplifies the sharing of code to a team. Many teams opt to go the [GitHub route](https://thenewstack.io/what-github-pull-requests-reveal-about-your-teams-dev-habits/) but there might be an occasion when you need to spin up a quick repository that is only available to those team members working on your LAN.

When you need to deploy a Git repository on your LAN and you need to give other team members access to it, the goal is to do it quickly and securely. Thanks to git and Secure Shell (SSH), this isn’t nearly as challenging as you might think. And although this setup might not be an option for team members who work outside of your LAN, it’s great for a temporary repository offered to those within your company network.

How does it work? Let me show you.

## What You’ll Need

To make this work, you’ll need the following:

- A Linux machine with Git installed.
- An SSH key pair.
- A user with [sudo privileges](https://thenewstack.io/linux-understand-sudo-to-rule-your-server/) (if the minimum requirements aren’t installed).

That’s it. Let’s make some [Git magic](https://thenewstack.io/take-your-first-steps-with-git/).

## Installing Git

On the off-chance Git isn’t installed, here’s how you can take care of that:

- Ubuntu-based distributions – `sudo apt-get install git -y`
- Fedora-based distributions – `sudo dnf install git -y`
- Arch-based distributions – `sudo pacman -S git`

## Create a Git User and Add the Required SSH Keys

First, we’re going to be talking about a remote and local machine. The remote machine will be the machine that houses the repository and the local machine(s) will access the remote repository.

Once Git is installed, you’ll want to add a specific user for authentication (because you don’t want to give other team members your username and password) on any/all remote machines that will access the repository. To create the new user, issue the command:

sudo adduser git

Make sure to give the user a strong/unique password and answer the remaining questions.

Change to the new user with the command:

su git

You should now be in the Git user’s home directory. To make sure of that, issue the command:

cd

Create the necessary directory that will house the authorized keys with the command:

mkdir .ssh

Give that directory the necessary permissions with the following:

chmod 700 .ssh

Create the authorized_keys file with:

touch .ssh/authorized_keys

Give that file the necessary permissions with the following:

chmod 600 .ssh/authorized_keys

Next, head over to the local machine and view the contents of the `id_rsa.pub` file for the user who’ll be accessing the remote repository with the command:

cat /home/USER/.ssh/id_rsa.pub

Where USER is the Linux username that will be working with the git repository.

If you’ve not already created SSH keys, the command for this is:

ssh-keygen

Copy the contents of that file.

Go back to the remote machine and open the authorized_keys file with:

nano /home/git/.ssh/authorized_keys

In that file, paste the content from the id_rsa.pub file and save it.

So far so good.

## Creating the Repository

It’s now time to create the actual repository. On the remote machine, issue the command:

mkdir /home/git/repository

Note: You can name the directory anything you like.

Change into that directory with the command:

cd /home/git/repository

Next, create a directory for a project with the command:

mkdir my_project

Again, you can name the project directory whatever you like.

Change into that new directory with:

cd my_project

Initialize the bare repository with the command:

git init --bare --shared

The bare option creates a repository with no content and the shared option ensures multiple people will be able to access it.

All right.

Your repository is ready to be cloned from the remote machine to the local machine.

## Cloning the Repository

Go back to the local machine and clone the new repository with the command:

git clone git@SERVER:/home/git/repository/my_project

You should see the following in the output:

_Cloning into ‘my_project’…_  
_warning: You appear to have cloned an empty repository._

You can verify the remote repository with:

git remote -v

You should see something like this:

_origin git@192.168.1.166:/home/git/repository/my_project (fetch)_  
_origin git@192.168.1.166:/home/git/repository/my_project (push)_

Now that we’ve verified, let’s create a README file on the local machine. From within the my_project directory, issue the command:

nano README

You can add whatever information to that file you need. Once you’ve done that, save and close the file. Next, add the file to staging with:

git add --all

You’ll then need to create your first commit with:

git commit -m "Added README file"

Finally, push the changes to the remote repository with:

git push origin master

You should see the README file in the repository on the remote machine. If not, you might have to check out the master branch (on the remote machine) with the command:

git checkout master

Let’s try one more thing.

On the remote machine, open the newly added README file with:

nano /home/git/repository/my_project/README

Change the contents of that file (you can add your name or anything you like).

Save and close the file.

Place the file into staging with the following:

git add .

Add a commit with:

git commit -m "Edited README file"

Back on the local machine issue the command:

git pull

When the pull completes, you can view the contents of the README and see that the changes are present.

And that’s all there is to creating a LAN-accessible Git repository. Make sure you add the `id_rsa.pub` key from any local machine to the authorized_keys file on the repository machine for those who need access and you should be good to go.

 Created with Sketch.