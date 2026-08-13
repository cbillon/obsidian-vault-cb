---
link: https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db
byline: nonbeing
site: Gist
excerpt: "Simple deployment using git's post-receive hook. GitHub Gist: instantly share code, notes, and snippets."
twitter: https://twitter.com/@github
slurped: 2026-07-05T17:52
title: Simple deployment using git's post-receive hook
tags:
  - git
  - deploy
---

Also see: [https://gist.github.com/lemiorhan/8912188](https://gist.github.com/lemiorhan/8912188)

## Simple automated deployment using git hooks

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#simple-automated-deployment-using-git-hooks)

Here are the simple steps needed to push your local git repository directly to a remote (e.g. prod) server over ssh. This is based on [Digital Ocean's Tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-git-hooks-to-automate-development-and-deployment-tasks).

## Overview

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#overview)

You are developing in a working-directory on your local machine, let's say on the `master` branch. Usually people push code to a remote server like github.com or gitlab.com and pull or export it to a production server. Or you use GitHub's webhooks to send a POST request to a webserver to take appropriate actions such as cloning/checking out a branch on the remote (prod) server.

But here you could simply use a `bare` git repository on the production server and publish a branch of your choice (e.g. `master`) directly to that server. This remote repo on the server acts upon the push event using a 'git hook' (in this case, the `post-receive` git hook) to put the files into a deployment directory on your server. No need for any intermediary such as GitHub.

This creates a scenario where there is no middle-man, high security with encrypted communication (using ssh keys, only authorized people get access to the server) and high flexibility from using a shell script (in the `post-receive` hook) for the deployment.

## Prerequisites

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#prerequisites)

1. Know how to use GIT, ssh etc.
2. Have a local working-directory ready to deploy, with AT LEAST 2 commits in the `git log`.
3. Have SSH access to your server using private/public key

## TLDR: Procedure

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#tldr-procedure)

- Have the local workspace dir ready to deploy
    
- Create a directory on your remote server to receive the deployment (e.g. `/var/www/html`)
    
- Add a bare git repository on the remote server
    
- Add the `post-receive` hook (shell script) to the bare repository, make it executable
    
- Add the remote-repository as a 'git remote' to your local git repository
    
- Push to the production server, relax.
    

## 1. Have a local working-directory ready to push

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#1-have-a-local-working-directory-ready-to-push)

Nuf said. I assume we are working on master – but you could work on any branch. Ensure there are at least 2 commits.

## 2. Create a directory for deployment on the remote server

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#2-create-a-directory-for-deployment-on-the-remote-server)

ssh into your remote (e.g. production) server.

We'll assume that your username is `webuser` on `server.com` and you access it (for "security-through-obscurity" reasons) over port 234 instead of the usual 22:

```
$ ssh webuser@server.com -p234
$ mkdir ~/deploy-dir  
```

(Note: If you ssh in over the default ssh port of `22`, then you can also just use the regular `ssh webuser@server.com` instead of `ssh webuser@server.com -p22`, but let's assume you use port `234`)

## 3. Add a bare git repository on the remote server

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#3-add-a-bare-git-repository-on-the-remote-server)

Now we'll create a "bare" git repository – one that does not contain any working copy files. It only has the contents of the `.git` directory in a normal working copy such as `refs`, `hooks`, `branches` etc. Call it whatever you like, but for our purposes, let's call it `bare-project.git`:

```
$ git init --bare ~/bare-project.git
```

## 4. Add the post-receive hook into the bare git repo

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#4-add-the-post-receive-hook-into-the-bare-git-repo)

The `post-receive` hook is a shell script that is executed when the push from the local machine has been received. We will write this script so that it deploys the files into required deployment directory (`~/deploy-dir` in this example). The `post-receive` file is located at this path: `~/bare-project.git/hooks/post-receive`. It must be named exactly as **post-receive**. Normally, it is not present by default, so use a text editor such as `vim` to create and edit it.

The script we will use does check if the correct branch is being pushed (it won't deploy a `develop` branch, e.g.)

See the [post-receive](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#file-post-receive) file for details.

Ensure it is executable: `chmod a+x ~/bare-project.git/hooks/post-receive`

## 5. Add a remote to your local git repo

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#5-add-a-remote-to-your-local-git-repo)

Now we'll add the bare repository (on the remote server) to your local system as a 'git remote'. For this example, `prod` is what we'll call this remote. This could also be called "staging" or "live" or "test" etc if you want to deploy to a different system or multiple systems.

```
$ cd ~/path/to/working-copy/on-your-local-system/

# assuming you ssh over port 234 instead of 22, 
# otherwise if you use the default port (22), you can just omit the `:234` part of the following url:
$ git remote add prod ssh://webuser@server.com:234/home/webuser/bare-project.git
```

Make sure `bare-project.git` corresponds to the name of the bare repo you used in step 3.

## 6. Deploy!

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#6-deploy)

Now you can push the master branch to the remote server:

```
$ git push prod master
```

That's it.

You should see something like:

$ git push prod master

Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 948 bytes | 948.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Ref refs/heads/master received. Deploying master branch on server...
remote: Already on 'master'
To ssh://webuser@server.com:234/home/webuser/bare-project.git
   d2b8c82..ac8be14  master -> master

## 7. Roll-backs

[](https://gist.github.com/nonbeing/f3441c96d8577a734fa240039b7113db#7-roll-backs)

If you pushed a "bad" deployment to the remote server and need to roll it back, fret not! It's easy to roll-back to an earlier, "good" commit using a forced push:

```
# on your local machine

# assuming that HEAD~1 is the commit you want to roll-back to
$ git reset --hard HEAD~1   # or git revert if you prefer that

# force push to the remote server
$ git push -f prod master
```

Code du web Hook
note : le hook reçoit 3 info oldrev newrev ref
seul ref nous intéresse
git woork permet de recuprer le source d'un repo bare
```
#!/bin/bash

#for debugging
#set -ex

# the work tree, where the checkout/deploy should happen
TARGET="/home/webuser/deploy-dir"

# the location of the .git directory
GIT_DIR="/home/webuser/project.git"


BRANCH="master"

while read oldrev newrev ref
do
        # only checking out the master (or whatever branch you would like to deploy)
        if [ "$ref" = "refs/heads/$BRANCH" ];
        then
                echo "Ref $ref received. Deploying ${BRANCH} branch on server..."
                git --work-tree="${TARGET}" --git-dir="${GIT_DIR}" checkout -f ${BRANCH}
        else
                echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
        fi
done
```