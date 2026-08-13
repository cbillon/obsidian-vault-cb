---
tags:
  - cbm
---

[Versionnig and Depreciation changes](https://moodle.org/mod/forum/discuss.php?d=457946)

Version X.Y.Z
- We will still release every 6 months, in April and October
- 
The changes are:

- The first part of the version will now represent a Series
- The last release in each Series will be an LTS release
- The next major release after an LTS will mark the beginning of a new Series

**Depreciations**

the deprecation policy for Moodle LMS to align more closely with the LTS cycle. The changes are:

- Deprecations will now be aligned with the LTS release cycle, providing a clear timeline for when   deprecated features need to be updated or removed.
- Features deprecated before an LTS release will be removed in the release following that LTS.
 
 Dan Marsden I know the HQ team spent a lot of time looking into the various version naming options - what they have settled on here lining it up with the deprecation process makes quite a lot of sense to me and should simplify a number of things.  
  
One big advantage that might not be clear is that if a 3rd party plugin works in 5.0 it should also work fine in 5.3, but then may break in 6.0 - we also like the last release in the series being the LTS version too.

 [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5)
Each plugin can have its own versionning concept
Can't HQ impose a versionning concept for additionnal plugins if they were to be included in official database 

par [Davo Smith](https://moodle.org/user/view.php?id=201866&course=5)

The simple answer is, that Moodle does enforce a versioning concept on plugins - it's built-in to the [Moodle plugins](https://moodle.org/plugins "Auto-link") directory.

If you pull the data from there, you get a list of which released version of each plugin is suitable for any given version of Moodle, along with a link to [download](https://moodle.org/mod/glossary/showentry.php?eid=18&displayformat=dictionary "Glossary of common terms : download") it (there's a 7MB [JSON](https://moodle.org/mod/glossary/showentry.php?eid=11554&displayformat=dictionary "Glossary of common terms : JSON") file you can pull down with everything in it - that's what our company's internal tools use for managing 3rd-party plugins).

Some plugins may support and encourage people to download them from github, in which case feel free to do so.

I can state, for example, that none of my own plugins offer stable versions in github - you can download from there if you want, but the stable, supported versions are in the plugins directory. There is no need for me, as a plugin developer, to conform to any particular branching scheme, because github is just the place where the current development code is kept for the plugin.

 [Thorsten Bartel](https://moodle.org/user/view.php?id=2756565&course=5)

Sorry for going off on a tangent here, but I have a quick follow up because the "sub-optimal solution" of git submodules Visvanath mentioned is something we utilize as well.  
If there is a change log linked in the plugins directory and maybe even a VCS tag specified, we regularly interpret this to imply: "This commit in our git repository is identical to this version of the plugin in the directory."  
  
Is this not the case? Or what are examples where the release in the plugin directory differs in a meaningful way from the linked / referenced commit on GitHub?  
Maybe you're just covering your base in case not so git-versed people get lost and consequently create support noise where it could be prevented by redirecting to a uniform source?  
  
Just as an example: [https://github.com/davosmith/moodle-checklist/commits/4.1.0.2](https://github.com/davosmith/moodle-checklist/commits/4.1.0.2)  
Everything in the plugins directory and on GitHub implies (to me) that the commit I linked is identical to the plugin version uploaded to and published in the plugins directory.

par [Neill Magill](https://moodle.org/user/view.php?id=1457891&course=5)

If you were to look at most of our plugins you would not find a commit or tag relating to a specific version number.

par [Davo Smith](https://moodle.org/user/view.php?id=201866&course=5)

Often new code is merged into the branch in github and then I wait to see results of the automated tests to make sure I've not accidentally broken something in one of the other branches (or with a certain PHP version).  
  
I can also sometimes fix a bug in github and ask the reporter to confirm that the fix has solved the problem (and not immediately introduced some other issues).  
  
In both of those cases, I don't send the new code to the Moodle plugins directory until I'm (more!) confident that I've not caused a problem with the updated code.  
  
Much of the time the code in github and the plugins directory is identical (especially as I don't make that many changes outside of Moodle major releases), but there are windows of time when github contains code that is more likely to have issues.  
  
During my work time, we carefully manage our branches and have a QA/UAT process whilst moving between branches (plus we don't have to worry about supporting multiple Moodle versions at once, as each customer we are writing the code for is only on a single Moodle version). In my spare time, when I maintain my own plugins, I'm not really willing to put in the extra time required for that level of branch management.


 [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5)
To bring that to an end I try to summarize the outcome:  

- The Moodle HQ, as the maintainer of the [plug-in database](https://moodle.org/plugins/) doesn't impose any Git workflow(s) for the plugins there. They expect the plug-in developers to submit zip files instead. Those are the ones which are shown under each plugin-database/plugin-name/versions and offered to downloaded or picked up by the Moodle front-end plug-in installer yoursite/admin/tool/installaddon/.  

- As a result the developers of each plug-in is free to follow whatever Git workflow convenient for them. Means, there are a multitude of workflows. 

Curiously, almost all plug-ins have a GitHub link published on its plug-in database, although there is no common understanding what that GitHub repo really is.

  

So the net result is, the system administrators (OS level, mainly Unix shell) have to either run a different set of Git commands for each plug-in or give up Git entirely and fall back to the zip archives - which is a step backward compared to the forward step Moodle made over a decade ago to [move from Zip files to Git](https://docs.moodle.org/20/en/Git_for_Administrators#Obtaining_the_code_from_Git).

 The recent discussions resulted in the HQ to Encourage plugin developers to have one branch per Moodle series.
