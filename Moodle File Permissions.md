---
tags:
  - moodle
  - configuration
  - permission
---
[Moodle docs](https://docs.moodle.org/35/en/Security_recommendations)
Depending on your server set-up there are two different scenarios:

1. You are running Moodle on your own dedicated server.
2. You are running Moodle on a shared hosting environment.

In the sections below, you are required to use the web service user account and group to set the permissions, so you need to know them. This can vary quite a bit from server to server but if this feature has not been disabled in your server, you can go to [http://your.moodle.site/admin/phpinfo.php](http://your.moodle.site/admin/phpinfo.php) (logging in as admin), and then search for the line that reads 'User/Group', inside the 'apache' table. For example, I get 'www-data' for the user account and 'www-data' for the group too.

### Running Moodle on a dedicated server

Assuming you are running Moodle on a sealed server (i.e. no user logins allowed on the machine) and that root takes care of the modifications to both moodle code and moodle config (config.php), then this are the most tight permissions I can think of:

1. moodledata directory and all of its contents (and subdirectories, includes sessions):

owner: apache user (apache, httpd, www-data, whatever; see above)
group: apache group (apache, httpd, www-data, whatever; see above)
permissions: 700 on directories, 600 on files

2. moodle directory and all of its contents and subdirectories (including config.php):

owner: root
group: root
permissions: 755 on directories, 644 on files.

If you allow local logins for regular users, then 2. should be:

owner: root
group: apache group (apache, httpd, www-data, whatever; see above)
permissions: 750 on directories, 640 on files.

Think of these permissions as the most paranoid ones. You can be secure enough with less tighter permissions, both in moodledata and moodle directories (and subdirectories).

### Running Moodle on a shared hosting environment

If you are running Moodle on a shared hosting environment, then above permissions are probably wrong. If you set 700 as the permission for directories (and 600 for files), you are probably denying the web service user account access to your directories and files.

If you want to tighten your permissions as much as possible, you will need to know:

1. the user account and the group the web service is running under (see above).
2. the owner of the directories/files of both moodledata and the moodle directory (this should normally be your client user account), and the group of the directories/files. You can usually get this information from the file manager of your hosting control panel. Go to the moodle folder and pick any directory or file and try to view/change the permissions, owner and group of that file. That would normally show the current permissions, owner and group. Do the same for the moodledata directory.

Then, depending on the following scenarios you should use a different set of permissions (listed from more secure to less secure) for your moodledata directory:

1. if the web service account and the owner of the directories/files is the same, you should use 700 for directories and 600 for files.
2. if the web service group and the group of the directories/files is the same, you should use 770 for directories and 660 for the files.
3. if none of the above, you will need to use 777 for directories and 666 for files, which is less secure but it is your only option. 707 and 606 would be more secure, but it might or might not work, depending on your particular setup.

In fact, you just need to set moodledata the permissions specified above, as all the directories and files inside are created by the web service itself, and will have the right permissions.

Regarding the moodle directory, as long as the web service user account can read the files plus read and execute the directories, that should be enough. There is no need to grant write permission to the web service account/group on any of the files or subdirectories. The only drawback is that you will need to create the config.php by hand during the installation process, as Moodle will not be able to create it. But that should not be a big problem.