---
link: https://www.drupal.org/node/2845379
excerpt: |-
  A required part of this site couldn’t load. This may be due to a browser
        extension, network issues, or browser settings. Please check your
        connection, disable any ad blockers, or try using a different browser.
slurped: 2026-08-16T10:16
title: Client Challenge
---
[link](https://www.drupal.org/node/2845379)
A required part of this site couldn’t load. This may be due to a browser extension, network issues, or browser settings. Please check your connection, disable any ad blockers, or try using a different browser.
# Provide optional composer integration but don't force users to understand how to use composer

One of the items mentioned during some of Dries' keynotes is that there's a danger of Drupal leaving beginners and site builder behind. A major step in the current march forward is Composer usage. While Composer is a great tool and should become the standard for building complex projects, it should not be _required_ to build a Drupal site unless a GUI is provided; additionally it should always be possible to download files from d.o, put them in the correct location (after extracting the archive) and have it work.

This should be seen as an opposition to [#2477789: Use composer to build sites](https://www.drupal.org/project/ideas/issues/2477789 "Status: Closed (duplicate)") becoming a requirement _unless_ a GUI is provided.

> it should always be possible to download files from d.o, put them in the correct location (after extracting the archive) and have it work.

We can wish for this all we want, but it's technologically impossible due to the way Composer is built.

Mixologic and I discussed this, and agreed that the right solution is to offer a rewritten Update Manager that embeds Composer (basically shipping with updater/ next to core/). That way people either use Composer, or they use the UI which runs Composer and then moves the changes into place. We also get core updates for free.  
[#2538090: Allow the Update Manager to automatically resolve Composer dependencies](https://www.drupal.org/project/drupal/issues/2538090 "Status: Closed (outdated)").