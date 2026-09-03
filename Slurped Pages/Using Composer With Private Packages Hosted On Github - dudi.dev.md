---
link: https://dudi.dev/composer-private-packages-github-repository
excerpt: Learn how to create private php packages, host them on github and
  import them using composer.
slurped: 2026-08-27T19:38
title: Using Composer With Private Packages Hosted On Github | dudi.dev
---

Composer is a great tool to import open source php packages hosted on [packagist.org](https://packagist.org/).

If you are creating packages with in your organization and want to keep them private, You cannot host them on packagist.org. Composer provides private package hosting as a paid service on [packagist.com](https://packagist.com/).

If you can afford the private packagist hosting, I recommend to use private packagist instead of hosting your private composer packages on github. By purchasing private packagist, You can financially support the development of composer and hosting costs for packagist.org.

If you are a solo developer, or a startup who cannot afford private packagist, Follow this guide to host your private php packages on github for free and import them using composer.

This article will create a simple composer php package, Host it as a private repository on github and import it to a project using composer.

## 1. Create Composer Package

Creating a composer private package is exactly same as creating a public package. So go ahead and **follow the steps from 1 to 4** outlined in the article, [creating composer php packages](https://dudi.dev/create-composer-php-package).

> Only difference is in step 2, Instead of marking the repository as public, **Mark the repository as private**.

## 2. Authorize composer to access private github repository

Now that our package is created and hosted on a private repository on github, We need to authorize composer to access our private github repository. This will allow composer to connect to our private github repository and download our package.

There are two ways of authorizing composer to download our private package from github.

### Option 1: Authorizing composer using auth.json file

> This method is Recommended only for local development and not for production usage.

If we are trying to download a package from a url which is private, Composer will look for authorization credentials for the private url in a file called `auth.json`.

We need to create personal access token on github and add it to `auth.json` file. So, composer can connect to our private repository using the personal access token provided.

> Read only permissions are sufficient for this personal access token as it is used by composer to only download the package from our private github repository.

To create a personal access token on github,

- go to [https://github.com/settings/tokens](https://github.com/settings/tokens)
- click on generate a new token button
- select `read:packages` checkbox
- click on `generate token` button
- copy the generated github token

Now that we have a github personal access token with read privileges to the private repository, We need to inform composer to connect to our private github repository using this personal access token. This can be done using the `auth.json` file.

create a new file called `auth.json` in your project root directory and add the following code to it.

```
1{2    "github-oauth": {3        "github.com": "your-github-token"4    }5}
```

replace `your-github-token` with your newly created github personal access token.

> You should never commit this file to github. Doing so will give unauthorized users access to your github repositories if the token is compromised.

We have our `auth.json` file all setup. By default, Composer will try to download packages from [packagist.org](https://packagist.org/). Since, Our packages are private packages hosted on github, They won't be available on packagist.org to download. We need to instruct composer about which github repository to look for inorder to find the package it is trying to download.

This can be done using the `repositories` array in your project `composer.json` file.

Open `composer.json` file in your project and add the following code to it.

```
1"repositories": [2    {3        "type": "vcs",4        "url": "https://github.com/your-github-username/your-repository-name"5    }6]
```

By adding the above code, We are instructing composer to look for the package in our private repository hosted on the specified url.

### Option 2: Authorizing composer using SSH key

The most recommended and secure way of authorizing composer with github is using ssh keys. Before authorizing composer using SSH key, We need to create an ssh key **on the machine where we are going to run composer**.

- Login to your server(or local machine) and run the following command to create an SSH key

```
1cd ~/.ssh/ && ssh-keygen -t rsa -b 4096 -C "[email protected]"
```

- Replace `[[email protected]](https://dudi.dev/cdn-cgi/l/email-protection)` with the **email address associated with your github account**.
- When you are prompted to enter a custom file name, Enter a file name(example : github_ssh).
- When you are prompted to enter a passphrase, Hit enter to ignore passphrase.
- create a new file named `config` in your `.ssh` directory by typing the below command.

```
1sudo nano ~/.ssh/config
```

- Add the following contents to the file and save it

```
1Host *2    Hostname github.com3    User git4    IdentityFile ~/.ssh/github_ssh
```

> Make sure to replace `github_ssh` with the file name you gave while creating the ssh key.

- Run `cat ~/.ssh/github_ssh.pub`(replace github_ssh with your ssh key file name). You will see the SSH key contents similar to below.

```
1ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+HvRnxwPJyUiUO/UCPKrW6mFPgJF8LxsC2l2bBePtn+UDv4Xy+eMJRgG5fbaqy2i0tvP+7T7bjVWCXJGIYunPbH978H4jrebF6Ts+dsgel4+ALf33z0nb9oaxCQF6V+T75hPgYp+JMOl8yZZMGLN3GPadE2ye2/lskJXzYjlHyjAE6a0g+vrHmMjOULP4UrO+aHEA84f   [email protected]
```

- Copy the SSH key content as shown.
- Go to [https://github.com/settings/keys](https://github.com/settings/keys)
- Click on `New SSH Key` button beside ssh keys
- Give a title for your ssh key
- In the key text box, Enter the SSH key content you previously copied
- Click on `Add SSH Key` button.

By adding this SSH key to our github account, Our machine is authorized to make secure ssh connections to github server.

Now all that is left is instructing composer to look for the private package in our private github repository. This can be done by updating the `repositories` array in project `composer.json` file.

Open `composer.json` file in your project and add the following code to it.

```
1"repositories": [2    {3        "type": "vcs",4        "url": "[email protected]:your-github-username/your-repository-name.git"5    }6]
```

By adding the above code, We are instructing composer to look for the package in our private repository. Since, We already configured SSH access to our private github repository, Composer should be able to download our private packages without any issues.

## 3. Import Private Package

Now that we have successfully authorized composer to access our github private repository, and also instructed it to where to look for our private package, We can start importing our private packages same way as we normally import composer packages.

Import the private package by running

```
1composer require vendor/package-name
```

> `vendor/package-name` is the name you added to your `composer.json` file inside your package.

**Example:**

```
1composer require srinath/hello-world-package
```

## Troubleshooting

If you are having issues importing your github private packages using composer, Try the following.

- Run `composer clearcache` to clear composer cache.
- Check if you added correct github personal access token inside `auth.json` file(If using option 1 for authorization).
- Verify if you added correct SSH key to your github account(If using option 2 for authorization).
- Confirm there is atleast one active release in your package repository which is **not marked as pre-release**.
- Verify the package name inside `composer.json` file of your package matches with the package name you are trying to import.
- Check if your package name has any spaces, special characters and uppercase characters in it.