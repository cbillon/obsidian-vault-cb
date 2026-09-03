---
link: https://www.mikestreety.co.uk/blog/how-to-delete-a-git-branch/
byline: Mike Street
site: Mike Street
date: 2022-11-19T01:00
excerpt: Make sure you keep your branches tidy by deleting old ones
slurped: 2026-08-22T22:34
title: How to delete a Git branch
---

### By Mike Street

2022-11-19T00:00:00.000Z Posted on **19th November 2022**. **1 mins** reading time

Deleting branches is important to ensure old code doesn't hang around or the wrong thing doesn't get merged

**Note:** In the examples below `pow` will be used for the branch name

## Delete a local branch

If you have a local branch you want to delete you can run

```
git branch -d pow
```

If your branch **has not been merged** into your **current branch** you need to change the `-d` to a capital:

```
git branch -D pow
```

## Delete a remote branch

If you wish to delete the branch via command line, you have to "push" it with a colon (`:`) preceding the name

```
git push origin :pow
```

## Updating your local branch data

If you deleted your remote branch from another computer (or via the website if on Github/Gitlab), you might find it is still listed when running a `git branch -a`. This means your local Git repository thinks the branch still exits and could cause conflicts if you try to create a branch of the same name.

To remove these, you can fetch with an additional `--prune` parameter

```
git fetch origin --prune
```

**Tip:** If you want it to always prune when you do a `git fetch origin`, you can set this as a global setting:

```
git config --global fetch.prune true
```

[View this post on Github](https://github.com/mikestreety/mikestreety/tree/main/app/content/blog/2022/2022-11-19-how-to-delete-a-git-branch.md "./app/content/blog/2022/2022-11-19-how-to-delete-a-git-branch.md")

[](https://fed.brid.gy/)

## You might also enjoy…

- [Turn any page into a JSON API with Cloudflare Workers](https://www.mikestreety.co.uk/blog/turn-any-page-into-a-json-api-with-cloudflare-workers/ "Turn any page into a JSON API with Cloudflare Workers")
    
    2022-12-10T00:00:00.000Z 10th December 2022
    
    ### [Turn any page into a JSON API with Cloudflare Workers](https://www.mikestreety.co.uk/blog/turn-any-page-into-a-json-api-with-cloudflare-workers/)
    
    Using the HTMLRewriter and Cloudflare workers, you can turn any webpage into a JSON endpoint
    
- [Use MinIO to cache Gitlab assets and leverage distributed runner caching](https://www.mikestreety.co.uk/blog/use-minio-to-cache-gitlab-containers-and-runners/ "Use MinIO to cache Gitlab assets and leverage distributed runner caching")
    
    2022-11-17T00:00:00.000Z 17th November 2022
    
    ### [Use MinIO to cache Gitlab assets and leverage distributed runner caching](https://www.mikestreety.co.uk/blog/use-minio-to-cache-gitlab-containers-and-runners/)
    
    Gitlab can utilise MinIO (S3 replacement) for caching built images, packages, uploads and for runner caches
    

![Mike Street](https://www.mikestreety.co.uk/assets/img/mike.jpg)

#### Written by

Mike is a CTO and Lead Developer from Brighton, UK. He spends his time writing, cycling and coding. You can find Mike on [Mastodon](https://hachyderm.io/@mikestreety).