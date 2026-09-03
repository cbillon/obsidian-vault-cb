---
link: https://www.mikestreety.co.uk/blog/backup-gitlab-data-locally/
byline: Mike Street
site: Mike Street
date: 2018-03-12T01:00
excerpt: I've recently moved all of my git repositories to Gitlab, this blog
  post walks through a script I have written to clone all of my repositories
  locally as a backup.
slurped: 2026-08-22T22:45
title: Backup Gitlab data locally
---

### By Mike Street

2018-03-12T00:00:00.000Z Posted on **12th March 2018**. **4 mins** reading time

For a while, I hosted my own version of [Gitlab](https://gitlab.com/) and, although I owned all the data, the server I had was not powerful enough for the install and would regularly crash and need rebooting. Moving my data across to the main Gitlab site gave me peace of mind that my data was always accessible no matter what time, however, I had less control over the data itself and no way of easily backing it up.

Below is a script is written in PHP which backs up my repositories for me. It is running on a small server I have at home which downloads the data onto a NAS. This NAS has two hard-drives which are mirrored.

The script does the following tasks:

- It connects via the Gitlab API and retrieves all of the repos I own
- It then finds all the groups I am a member of and the repos of those groups
- Once collated and merged, it loops through each one
- If it doesn't exist locally, it clones it down as a bare repository
- If it exists already, it updates the repo

By doing this, if Gitlab closes down tomorrow, I have the data. As I rarely do side projects and updates, the script is set to run monthly. I consider that any work I have done in the last month will be backed up somewhere else (on a live/dev server somewhere).

As this script uses the API, it doesn't need updating each time I add a project to Gitlab itself! I've included the code below with plenty o' comments to help you through.

The first line is a `shebang`. This allows you to just run the file without to specify `php` when running on the command line.

```
#!/usr/bin/php
<?php
	// Function to make any Gitlab cURL requests easier and it just requires
	// the path to be passed in. No need to use a full API composer package
	// for these commands
	function gitlabCurl($path) {
		// Get your token from Gitlab.com (or your own Gitlab install)
		$header = array('PRIVATE-TOKEN: XXXX');

		// Make a cURL request to Gitlab. if you're not using gitlab.com update
		// the URL below to match
		$ch = curl_init('https://gitlab.com/api/v4/' . $path);
		curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
		curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
		curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
		$result = curl_exec($ch);
		curl_close($ch);

		// Lopp through your output setting the project ID as the key - this
		// ensures you can merge arrays without fear of overwriting anything.
		// It also sorts them into a logical order
		$ouput = array();
		foreach(json_decode($result, true) as $p) {
			$output[$p['id']] = $p;
		}

		// Parse returned list to an array
		return $output;
	}


	// Collate all the projects you personally own (don't forget to update the
	// username)
	$projects = gitlabCurl('users/mikestreety/projects');

	// Get all of the groups the authenticated user is a member of
	$groups = gitlabCurl('groups');
	foreach ($groups as $group) {
		// Get the projects from each group and merge with the previous projects
		$projects = array_merge(
			$projects,
			gitlabCurl('groups/' .  $group['id'] . '/projects')
		);
	}

	// Set a backup location for your repos
	$location = __DIR__ . '/backups/';

	// Shuffle projects so if script fails, different projects get backed up
	// at different times
	shuffle($projects);

	// Loop through the projects
	foreach ($projects as $p) {
		// output the name, so if it fails you know which project it was on
		echo $p['name'] . PHP_EOL;

		// use the namespace to set a folder path, this keeps projects in
		// alphabetical order in according to group
		$folder = $location . str_replace('/', '-', $p['path_with_namespace']);

		// Check if the repo exists already
		if(file_exists($folder)) {
			// If it does, cd and pull everything (branches, tags etc)
			exec('cd ' . $folder . '&& git remote update');
		} else {
			// If not, clone as a bare/mirror repo in the folder
			exec('git clone --mirror ' . $p['ssh_url_to_repo'] . ' ' . $folder);
		}
	}
```

[View this post on Github](https://github.com/mikestreety/mikestreety/tree/main/app/content/blog/2018/2018-03-12-backup-gitlab-data-locally.md "./app/content/blog/2018/2018-03-12-backup-gitlab-data-locally.md")

[](https://fed.brid.gy/)

## You might also enjoy…

- [Why we're not dropping Slack for Google Hangouts Chat...yet.](https://www.mikestreety.co.uk/blog/why-were-not-dropping-slack-for-google-hangouts-chat-yet/ "Why we're not dropping Slack for Google Hangouts Chat...yet.")
    
    2018-03-19T00:00:00.000Z 19th March 2018
    
    ### [Why we're not dropping Slack for Google Hangouts Chat...yet.](https://www.mikestreety.co.uk/blog/why-were-not-dropping-slack-for-google-hangouts-chat-yet/)
    
    [**Liquid Light**](https://www.liquidlight.co.uk/blog/why-were-not-dropping-slack-for-google-hangouts-chat-yet/)
    
    At the beginning of this month, Google announced they were rolling out Hangouts Chat to all users of Google Suite over a week or so. Our suite was finally updated ...
    
- [How I wrote a book; the writing process from one of our Developers](https://www.mikestreety.co.uk/blog/how-i-wrote-a-book-the-writing-process-from-one-of-our-developers/ "How I wrote a book; the writing process from one of our Developers")
    
    2018-03-08T00:00:00.000Z 8th March 2018
    
    ### [How I wrote a book; the writing process from one of our Developers](https://www.mikestreety.co.uk/blog/how-i-wrote-a-book-the-writing-process-from-one-of-our-developers/)
    
    [**Liquid Light**](https://www.liquidlight.co.uk/blog/how-i-wrote-a-book-the-writing-process-from-one-of-our-developers/)
    
    In 2017, one of our front-end developers wrote and released a book about the JavaScript library; Vue.js. This blog post explains the process he went through to get the book from his mind to print.
    

![Mike Street](https://www.mikestreety.co.uk/assets/img/mike.jpg)

#### Written by

Mike is a CTO and Lead Developer from Brighton, UK. He spends his time writing, cycling and coding. You can find Mike on [Mastodon](https://hachyderm.io/@mikestreety).