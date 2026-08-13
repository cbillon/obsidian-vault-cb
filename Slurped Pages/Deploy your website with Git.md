---
link: https://peteris.rocks/blog/deploy-your-website-with-git/
byline: Pēteris Ņikiforovs
site: peteris.rocks
excerpt: How to deploy websites to different environments with git
slurped: 2026-07-05T11:39
title: Deploy your website with Git
tags:
  - git
  - deploy
---

Goal: deploy your website by running a simple command on your computer.

Before pushing changes to production you'd like to check that everything looks alright. Let's suppose that the staging server where you preview your changes is also hosted on the same production server. This works well with simple or static websites.

```
/var/www/domain.com
/var/www/domain.com/production
/var/www/domain.com/staging
/var/www/domain.com/repository.git
```

## 

Remote git repository setup

Create an empty git repository

```
mkdir -p /var/www/domain.com/repository.git
cd /var/www/domain.com/repository.git
git init --bare
```

Add a script, called a hook, that'll be executed when there's incoming changes

```
cat > hooks/post-receive <<END
#!/bin/bash

set -e

BASE=/var/www/domain.com

while read oldrev newrev refname
do
    branch=$(git rev-parse --symbolic --abbrev-ref $refname)

    BRANCH_DIR="$BASE/$branch"
    BRANCH_TEMP_DIR="$BASE/$branch.temp"

    rm -rf $BRANCH_TEMP_DIR
    mkdir -p $BRANCH_TEMP_DIR
    cd $BRANCH_TEMP_DIR

    git --work-tree=$BRANCH_TEMP_DIR --git-dir=$BASE/repository.git checkout $branch -f

    # run your custom commands here

    rm -rf $BRANCH_DIR
    mv $BRANCH_TEMP_DIR $BRANCH_DIR
done
END
```

Make sure it can be executed

```
chmod +x hooks/post-receive
```

## 

Local setup

Let's suppose all your changes are on the `master` branch.

Add a new remote to your repository

```
git remote add server ssh://user@host/var/www/domain.com/repository.git
```

So then just do

```
git push server master:staging
```

If everything looks alright

```
git push server master:production
```

## 

Different servers

Your hook is very simple because you're probably going to use just one main branch e.g. `master`.

```
cat > hooks/post-receive <<END
#!/bin/bash
set -e
BASE=/var/www/domain.com
cd $BASE
git --work-tree=$BASE/public --git-dir=$BASE/repository.git checkout master -f
# run your custom commands here
END
```

Add remotes for each server

```
git remote add production ssh://user@production-host/var/www/domain.com/repository.git
git remote add staging ssh://user@staging-host/var/www/domain.com/repository.git
```

To push changes to production

```
git push production
```

To push changes to staging

```
git push staging
```