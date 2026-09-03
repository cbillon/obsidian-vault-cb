---
link: https://www.gizra.com/content/private-composer-repos-in-ddev/?utm_source=thedroptimes
site: Web Strategy, Design and Development | Gizra
excerpt: How to use private composer repos for Drupal projects in DDEV
twitter: https://twitter.com/@gizra_drupal
slurped: 2026-08-23T14:26
title: Private Composer Repos Using DDEV
---

We do not usually make use of private composer repos. The reason is simple, all our private code lives inside a single repo.

But sometimes, we need to re-use a project for multiple sites, and we still want to keep the code private. In those cases, a private composer repo makes sense.

## The Composer Part

To use a private composer repo you need two things. Let’s suppose our private project is called `reusable_code` and lives in `github.com/Gizra/reusable_code` (you won’t see it… It is private).

First, add the repository to `composer.json`

```
"repositories": [
  {
      "type": "vcs",
      "url":  "gizra-github:Gizra/reusable_code.git"
  },
],
```

Note the `gizra-github` alias in the URL, we will use this later.

And then add the repo path to the required section.

```
require: {
  "gizra/reusabled_code": "dev-master",
}
```

## The Authentication

Next, you need to allow composer to connect the private repository without hardcoding your password.

You can use SSH keys for this task. Assuming you don’t have one:

```
ssh-keygen -o -a 256 -t ed25519 -C "[email protected]"
```

This will generate two files that are a pair of Private and Public keys. Something named `~/.ssh/id_ed25519` and `id_ed25519.pub` or similar.

Then you can instruct SSH that the `gizra-github` is actually an alias for github.com.

In your `~/.ssh/config` add this section:

```
Host gizra-github
  HostName github.com
  User git
```

If you [configured GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) with your public key, and if you run `composer require gizra/reusable_code -n`, you will be able to download a private repo to your codebase.

## The DDEV Complexity

If you try to run `ddev composer require gizra/reusable_code -n`, it will fail.

The reasons are:

1. DDEV does not know what `gizra-github` means.
2. DDEV does not know about your private/public key (yet).

To fix 1, you will need to add the following config to `.ddev/homeadditions/.ssh/config`.

```
Host gizra-github
  HostName github.com
  User git
  IdentityFile=~/.ssh/id_ed25519
```

Also copy your `~/.ssh/id_ed25519.pub` into `.ddev/homeadditions/.ssh/id_ed25519.pub` and remember to run `ddev restart`.

To fix 2, you will need to run `ddev auth ssh`.

After this setup you will be able to run `ddev composer require gizra/reusable_code -n` without any auth issues.

## Working with a Team

So far the described approach works, if you are the only one working in the project. The SSH keys are **your** keys and you probably don’t want to share them with anyone else.

If you have more people in your team, then you need to either create a specific SSH key for the team, or better yet, a GitHub deploy key.

Deploy keys are basically an SSH key pair, but without write access, and are only valid for a particular project.

So create a new SSH key pair, [add it as a deploy key to the private repo](https://docs.github.com/en/developers/overview/managing-deploy-keys), and then share the private key with your team.

In our case, the final project layout is:

Put the public deploy key here: `.ddev/homeadditions/.ssh/deploy_key.pub` Use the following ssh config: `.ddev/homeadditions/.ssh/config`

```
Host gizra-github
  HostName github.com
  User git
  IdentityFile=~/.ssh/deploy_key
```

Then run `ddev auth ssh` and use `ddev composer install` as usual.

## The CI Part

If you have some Continuous Integration system in place, you will need to figure out a way to send the private key into the CI system.

You may want to check how to do this for example in TravisCI: [https://docs.travis-ci.com/user/encryption-keys/](https://docs.travis-ci.com/user/encryption-keys/)

## Alternatives

If you think all this is a lot of complexity, as it adds an extra step for team members (the need to run `ddev auth ssh`) before being able to run `ddev composer install`, you are probably right.

If you are willing to make this a bit less secure, you can opt to add to your project the public, **and** the private key of the shared repo. After all, if you or the CI tool have access to main repository, it is ok to have access to the private shared repo as well.

This simplifies things as you don’t authenticate via ddev every time. It also simplifies the key management in the CI environment. The private key does not need to be encrypted and decrypted anymore.

However, you’ll need to add some extra security measures you need to keep in mind. Like rotating the keys from time to time, if you gave access to external collaborators that aren’t working on your team anymore.

You still need however to add the mentioned configurations to `.ddev/homeadditions/.ssh/config` and perform an extra step in your `.ddev/config.yaml` to give the right permissions to the private SSH key.

```
hooks:
  post-start:
  - exec: chmod 600 ~/.ssh/deploy_key
```

And your `.ddev/homeadditions/.ssh` directory would look like this:

```
$ ls -l .ddev/homeadditions/.ssh/

-rw-r--r-- 1 user user 416 may 19 09:47 config
-rw-r--r-- 1 user user 419 may 19 09:47 deploy_key
-rw-r--r-- 1 user user 107 may 19 09:47 deploy_key.pub
```

Note that both, the public and the private deployment keys are now part of the `.ddev/homeadditions/.ssh` directory.

## FAQ

- Why do we need to use an SSH config?

This is because, by default, composer will not use `git` as a user when pulling the repo. This is the reason behind: `User git`.

- Why do we use the public key instead of the private key in the config?

The call to `ddev auth ssh` already passes the private key to the containers. This is a more secure way than copying the private key inside the container. This is one of the [reasons why ddev-ssh-agent exists](https://github.com/drud/ddev/pull/2224#issuecomment-620179053).

You can check this by running `ddev ssh` and then `ssh-add -L` after running `ddev auth ssh`.

- Why do we need an alias for the github.com URL?

While you can use github.com directly in the `~/.ssh/config`, this will force all git commands to run as `git` while interacting with github.com. This may introduce some conflicts in your workflow.

- Is this guide only relevant to GitHub?

No, [GitLab](https://docs.gitlab.com/ee/user/project/deploy_keys/) also provides deploy keys.

## Bonus

1. When running `ddev composer install` if you encounter the following error:
    
    ```
    Load key "~/.ssh/deploy_key": invalid format
    ```
    
    This can be caused by having `git config core.autocrlf` set to `true`. If you get this message, set `git config core.autocrlf false` and restart ddev, and try running ddev composer installer again. It may be worth deleting the offending key file located in `.ddev/homeadditions/.ssh/deploy_key` then checking it out of git again before restarting ddev.