---
link: https://amitmerchant.com/never-run-composer-update-on-your-server/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2024-06-06T02:00
excerpt: Povilas Korop recently shared an interesting (yet important) tip regarding sane use of Composer if you’re used to working with Composer on your server.
slurped: 2025-10-13T11:40
title: Never run composer update on your server
tags:
  - composer
---

Povilas Korop recently [shared](https://twitter.com/PovilasKorop/status/1798299978265243957) an interesting (yet important) tip regarding sane use of Composer if you’re used to working with Composer on your server.

So, let’s say you need to update Composer packages on your server for some reason, you would make the mistake of running `composer update` on the server. This will update the `composer.lock` file with the new packages and you’re left with a dirty repository on your server. And you stuck in this limbo where you might not want to commit the `composer.lock` file from the server because usually you would often pull the changes on a live server rather than push them.

Apart from this, `composer update` is a little slow in updating dependencies. So, it might slow down your CI/CD pipelines.

> So, the ideal way here would be to run `composer update` on your local machine. This will update the `composer.lock` file with the new dependencies and you can then [push the changes to your server](https://amitmerchant.com/why-you-should-always-commit-the-composer-lock-file/).

You can then run `composer install` to install the new updates on the server. This will install the Composer dependencies based on the `composer.lock` file without updating it further. So, you’ll have a clean repository on your server, and on top of this, the dependencies will be installed relatively fast.