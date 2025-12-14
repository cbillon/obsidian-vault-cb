---
tags:
  - moodle
  - permission
---
https://moodle.org/mod/forum/discuss.php?d=468396

You can solve the mystery yourself starting from the first principles.

They are:
1. The web server processes belong to a certain user and a group. For example in Apache2 under Debian Linux, they are in the /etc/apache2/envvars. export APACHE_RUN_USER=www-data, export APACHE_RUN_GROUP=www-data.

2. Those web server processes need read access to the Moodle code, in your case /var/www/moodle/ (and everything below that).
 
Yes, only _read_ access.

3. Those web server processes need write access to the "moodledata" directory, in your case /var/www/moodledata/.
 
I repeat, _write_ access (and obviously read access too).
 
Usually you give Moodle an empty moodledata directory and from there on Moodle manages moodledata/ on its own.
 
I know, you know this, but to eliminate a common misconception: The "everyone" group in Unix, what you call the World, is not the 8 billion world population nor the 5 billion of them having web access, rather all the other shell user accounts of the server that runs Moodle (more accurately, the web server that powers it).

en réponse à :

We have **Moodle 4.4.6+ (Build: 20250214)** installed on a stand alone Linux [server](https://moodle.org/mod/glossary/showentry.php?eid=30&displayformat=dictionary "Glossary of common terms : server") (no other users/apps).  
The scan has identified world writable files as a security risk.  
  
After reviewing the links below, I've decided to modify the files and directories

- **References**:  
    [https://docs.moodle.org/404/en/Security_recommendations#Running_Moodle_on_a_dedicated_server](https://docs.moodle.org/404/en/Security_recommendations#Running_Moodle_on_a_dedicated_server)  
    [https://docs.moodle.org/404/en/Installing_Moodle#Download_and_copy_files_into_place](https://docs.moodle.org/404/en/Installing_Moodle#Download_and_copy_files_into_place)  
    [https://docs.moodle.org/404/en/Installation_quick_guide#Install_Moodle_code](https://docs.moodle.org/404/en/Installation_quick_guide#Install_Moodle_code)  
      
    
- Ran the following commands on my server:
    
    - Change permissions for moodle app code:  
        `sudo find /var/www/moodle -type d -exec chmod 0755 {} \;`  
        `sudo find /var/www/moodle -type f -exec chmod 0644 {} \;`
        
        - **0755 (dirs):** Allows the owner full access and group/others to read and enter the directory.
            
        - **0644 (files):** Owner can read/write, group and others can read — no write access for anyone else.
            
    - Change permissions for moodle app data:`sudo find /var/www/moodledata -type d -exec chmod 0750 {} \;`  
        `sudo find /var/www/moodledata -type f -exec chmod 0640 {} \;`
        
        - **0750 (dirs):** Allows the owner full access and group members to read/enter.
            
        - **0640 (files):** Owner can read/write, group can read — no access for "others."  
              
            
        

After doing this, existing files and directories were no longer world writable, but moodle continued to produce new world writable files.  
So I then made the following changes to moodle’s config file, as well as modifying apache’s `umask` setting.

- **/var/www/html/config.php**
    - Originally:

- - - `$CFG->directorypermissions = 02777;`
            

- - Changed to:

- - - `$CFG->directorypermissions = 02750;`
            
            - 02750 - [Apache](https://moodle.org/mod/glossary/showentry.php?eid=9851&displayformat=dictionary "Glossary of common terms : Apache") + group can read/exec, Some group-based read access is needed
                

- - Also, added this near the top of config.php:

- - - `umask(0027);`
            
            - This ensures that Moodle uses the correct umask even if the server is misconfigured.
                

-  **/etc/apache2/envvars**
    - Add this at the end (or edit if it already exists):

- - - `umask 0027`
            
            - Any new files or directories created by Apache processes will have permissions that do not allow world access.
                

  
However, moodle is still creating world writable files (most recent identified was **/var/www/moodledata/localcache/di/CompiledContainer.php (-rw-rw-rw-)**  
  
While looking into this, I've come across one or two posts claiming that the entire moodledata directory _should_ be world writable.  
I don't think this is correct based on the other links I have shared above (and many more that I read regarding "hardening" moodle).  
  
So my questions now are as follows:

1. Can someone confirm if this directory should be world writable?
    1. If yes, please explain how moodle protects the files in there and the server from malicious behaviour (I will need this information to submit and exception request to my security team)
    2. If no, how can I prevent moodle from creating new world writable files?  
          
        
2. Can someone point me to official documentation that deals with directory and file creation permissions?
    1. I followed an assortment of guides when installing, but mostly the [Step-by-step Installation Guide for Ubuntu - MoodleDocs](https://docs.moodle.org/404/en/Step-by-step_Installation_Guide_for_Ubuntu). It briefly mentions changing permissions after install, but doing that (as I did) still results in world writable files

