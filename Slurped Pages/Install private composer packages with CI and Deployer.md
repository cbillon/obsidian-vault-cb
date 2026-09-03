---
link: https://www.mikestreety.co.uk/blog/install-private-composer-packages-with-ci-and-deployer/
byline: Mike Street
site: Mike Street
date: 2023-08-03T02:00
excerpt: Using environment variables you can install private composer packages
  in Gitlab CI and pass them through to Deployer
slurped: 2026-08-22T22:25
title: Install private composer packages with CI and Deployer
---

2023-08-03T00:00:00.000Z Posted on **3rd August 2023**. **2 mins** reading time

Installing private composer packages can be a bit like Crufts - you sometimes have to jump through so many hoops and tunnels and all you get at the end is a belly rub.

However you host your packages, the general theory is the same. I would advise finding a host which can act like a library endpoint rather than individual Git repos. Something like Github, Gitlab or Bitbucket can do this for you.

This post assumes you have a private composer repository host and are looking to access it using tokens for CI and deployment purposes. I'll be using Gitlab as my endpoint, but the code can be substituted for any other host.

Setting up authorisation locally can be done by running `composer config --auth`. This will make a local `auth.json` file in your repo (try not to commit this). If you use the same private packages across a few different sites, you can add `--global` to set the global auth file. Mine ended up looking something like the below

```
{
   "gitlab-domains":[
      "private.gitlab.com"
   ],
   "gitlab-token":{
      "private.gitlab.com": "gplat-token"
   }
}
```

- `private.gitlab.com` is the URL to my private Gitlab instance
- `gplat-token` is an access token generated with `api` access

This allows me locally install my private packages - but how do we use it in Gitlab?

## Gitlab CI

CI or not, an alternative way of passing in Composer authorisation details is setting a `COMPOSER_AUTH` environment variable. With Gitlab CI, setting a environment variable can be done using the [`variables` keyword](https://docs.gitlab.com/ee/ci/variables/)

```
variables:
  COMPOSER_AUTH: '{"gitlab-domains": ["private.gitlab.com"],"gitlab-token": {"private.gitlab.com": "gplat-token"}}'
```

You should be able to install your private packages. However, committing the URL and token is not a good idea. Fortunately, Gitlab has some [predefined variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) we can utilise.

First, create a CI/CD [variable in the UI](https://docs.gitlab.com/ee/ci/variables/#define-a-cicd-variable-in-the-ui) called `COMPOSER_TOKEN`.

We can then use it, along with the predefined `$CI_SERVER_HOST` variable to build up our COmposer Auth env variable

```
variables:
  COMPOSER_AUTH: '{"gitlab-domains": ["$CI_SERVER_HOST"],"gitlab-token": {"$CI_SERVER_HOST": "$COMPOSER_TOKEN"}}'
```

## Deployer

If you use [PHP Deployer](https://deployer.org/) to push your code live, it will need access to this auth variable in the target server to install your packages. This can be passed through with the `env()` function:

```
set('env', [
	'COMPOSER_AUTH' => getenv('COMPOSER_AUTH'),
]);
```