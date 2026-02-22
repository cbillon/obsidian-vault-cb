---
tags:
  - moodle
  - Installation
---
Step-by-step Installation Guide for Ubuntu
[Installation Guide](https://docs.moodle.org/501/en/Step-by-step_Installation_Guide_for_Ubuntu)
Contents
1 Development Database Installation
1.1 Update and install base packages
1.2 Install a Webserver
1.2.1 Option 1: Install Apache as your Webserver
1.2.2 Option 2: Install Nginx as your Webserver
1.3 Obtain Moodle code using git
1.4 Specific Moodle requirements
1.5 Create Database and User
1.6 Configure Moodle from the command line
2 Prepare Development Database for Production
2.1 Check Permissions
2.2 Set a root password on the database
2.3 Configure and enable a firewall
2.4 Convert http to the encrypted https protocol
2.4.1 Apache version
2.4.2 Nginx version
2.5 SSH keys
2.6 Remove SSH root access
2.7 Keep Ubuntu up to date
2.8 SQL Backups
2.9 Setting up antivirus on the server
Development Database Installation
This guide requires a Linux Ubuntu 24 server with sudo or root access. All commands on this page are text based so there is no need for a desktop with a graphical interface. Each step will tell you what you need to do and then give the commands that will accomplish that step. You can copy the entire coloured step or a line at a time but be careful with the word wrap, some commands will extend over multiple lines.

Apache or Nginx are gvien as options for a webserver. Install on or the other, not both. Both are good options, with Apache having good community support while Nginx may be better suited to busier sites.

Update and install base packages
Moodle requires a web server, database and the php scripting language as described in https://moodledev.io/general/releases/5.0#server-requirements.

# Set your domain name or IP address as a temporary variable as it will be needed several times in the installation
PROTOCOL="http://";
read -p "Enter the web address (without the http:// prefix, eg domain name mymoodle123.com or IP address 192.168.1.1.): " WEBSITE_ADDRESS

```
MOODLE_PATH="/var/www/html/sites"
MOODLE_CODE_FOLDER="$MOODLE_PATH/moodle"
MOODLE_DATA_FOLDER="/var/www/data"
sudo mkdir -p $MOODLE_PATH
sudo mkdir -p $MOODLE_DATA_FOLDER

```

# Refresh and download latest versions of all packages

```
sudo apt-get update && sudo apt upgrade -y 

```

# Get php-fpn and required php extensions using the package manager apt-get

```
sudo apt-get install -y php8.3-fpm php8.3-cli php8.3-curl php8.3-zip php8.3-gd php8.3-xml php8.3-intl  php8.3-mbstring php8.3-xmlrpc php8.3-soap php8.3-bcmath php8.3-exif php8.3-ldap php8.3-mysql

```

# Database and packgages required by Moodle
sudo apt-get install -y unzip mariadb-server mariadb-client ufw nano graphviz aspell git clamav ghostscript composer
Install a Webserver
Apache or Nginx are given as options for a webserver. Choose Option 1 or Option 2, not both. Both are good choices, with Apache having good community support while Nginx may be better suited to busier sites.

Option 1: Install Apache as your Webserver
# Install Apache and module for FastCGI protocol
```
sudo apt-get install -y apache2 libapache2-mod-fcgid

```
# Enable required Apache modules for PHP-FPM and rewriting
```
sudo a2enmod proxy_fcgi setenvif rewrite

```

# Create Moodle Apache site config with fallback and PHP-FPM socket
```
sudo tee /etc/apache2/sites-available/moodle.conf > /dev/null <<EOF
<VirtualHost *:80>
    ServerName $WEBSITE_ADDRESS
    ServerAlias www.$WEBSITE_ADDRESS

    DocumentRoot  $MOODLE_CODE_FOLDER/public

    <Directory $MOODLE_PATH>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
        DirectoryIndex index.php index.html

        # Enable fallback routing for URLs not matching files/directories
        FallbackResource /r.php
    </Directory>

    # PHP-FPM 8.3 FastCGI handler via Unix socket
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    # Log files
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOF
```

```
# Enable the new Moodle site and disable the default site if desired
sudo a2ensite moodle.conf
sudo a2dissite 000-default.conf

```


# Reload Apache to apply all changes
```
sudo systemctl reload apache2

```
# Enable and start PHP 8.3 FPM service


```
sudo systemctl enable --now php8.3-fpm

```
# Adjust PHP settings for both Apache and CLI

```
php_ini_apache="/etc/php/8.3/fpm/php.ini"
php_ini_cli="/etc/php/8.3/cli/php.ini"

sudo sed -i 's/^[[:space:]]*;*[[:space:]]*max_input_vars[[:space:]]*=.*/max_input_vars = 5000/' "$php_ini_apache"
sudo sed -i 's/^[[:space:]]*;*[[:space:]]*max_input_vars[[:space:]]*=.*/max_input_vars = 5000/' "$php_ini_cli"

sudo sed -i 's/^\s*post_max_size\s*=.*/post_max_size = 256M/' "$php_ini_apache"
sudo sed -i 's/^\s*post_max_size\s*=.*/post_max_size = 256M/' "$php_ini_cli"

sudo sed -i 's/^\s*upload_max_filesize\s*=.*/upload_max_filesize = 256M/' "$php_ini_apache"
sudo sed -i 's/^\s*upload_max_filesize\s*=.*/upload_max_filesize = 256M/' "$php_ini_cli"

```

# Reload PHP-FPM to apply PHP configuration changes
```
sudo systemctl reload php8.3-fpm

```

The Apache webserver is now installed. Go to "Moodle Code" to continue the installation.

Option 2: Install Nginx as your Webserver
Do not install Nginx if you have already installed Apache!

```
  sudo apt-get install -y nginx
```

# Set up the configuration file including the fallback required for the router
# Using tee allows the file to be written in a single (rather long command). This could also been have done with a text editor.
# Be sure to copy and paste entire block from "sudo" to "EOF"


```
sudo tee /etc/nginx/sites-available/moodle.conf > /dev/null <<EOF
server {
    listen 80;
    server_name $WEBSITE_ADDRESS www.$WEBSITE_ADDRESS;
    root $MOODLE_CODE_FOLDER/public;
    index index.php index.html index.htm;

    location / {
        try_files \$uri \$uri/ /index.php?\$args /r.php;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+\.php)(/.+)\$;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        fastcgi_param PATH_INFO \$fastcgi_path_info;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOF
```
# Recognise the new config file

```
sudo ln -s /etc/nginx/sites-available/moodle.conf /etc/nginx/sites-enabled/moodle.conf
sudo systemctl reload nginx

```

# Make necessary changes to the php configuration required by Moodle
# Using sed finds and replaces text. This could have been done in a test editor


```
sudo sed -i 's/^;max_input_vars =.*/max_input_vars = 5000/' /etc/php/8.3/fpm/php.ini
sudo sed -i 's/^;max_input_vars =.*/max_input_vars = 5000/' /etc/php/8.3/cli/php.ini
sudo sed -i 's/^post_max_size =.*/post_max_size = 256M/' /etc/php/8.3/fpm/php.ini
sudo sed -i 's/^post_max_size =.*/post_max_size = 256M/' /etc/php/8.3/cli/php.ini
sudo sed -i 's/^upload_max_filesize =.*/upload_max_filesize = 256M/' /etc/php/8.3/fpm/php.ini
sudo sed -i 's/^upload_max_filesize =.*/upload_max_filesize = 256M/' /etc/php/8.3/cli/php.ini
sudo systemctl reload php8.3-fpm

```
Obtain Moodle code using git
git is a version control system that is the easiest way to download the latest Moodle code and unpack it into a directory. It also makes keeping up to date with patches much easier.

 
# Clone to the Moodle code folder

```
sudo git clone -b v5.1.0 https://github.com/moodle/moodle.git $MOODLE_CODE_FOLDER
sudo chown -R www-data:www-data $MOODLE_CODE_FOLDER
cd  $MOODLE_CODE_FOLDER
CACHE_DIR="/var/www/.cache/composer"
sudo mkdir -p "$CACHE_DIR"
sudo chown -R www-data:www-data "$CACHE_DIR"
sudo chmod -R 750 "$CACHE_DIR"
sudo -u www-data COMPOSER_CACHE_DIR="$CACHE_DIR" composer install --no-dev --classmap-authoritative
sudo chown -R www-data:www-data vendor
sudo chmod -R 755 $MOODLE_CODE_FOLDER

```

Specific Moodle requirements

Moodle needs to store files in a specific directory accessible only by the web server. It also needs to change some php configuration settings and set up a cron task. This could be done using a text editor but echo and sed, a text replacement tool, simplifies the process.

#  Create the moodledata directory outside your web server's document root
```
sudo mkdir -p $MOODLE_DATA_FOLDER/moodledata

```
# Set the webserver as the owner and group recursively ( for both the files and contents)
```
sudo chown -R www-data:www-data $MOODLE_DATA_FOLDER/moodledata

```
#  Set the  moodledata directory permissions so only the web server can read, write, and access them.
```
sudo find $MOODLE_DATA_FOLDER/moodledata -type d -exec chmod 700 {} \;

```

# Set the  moodledata file permissions so only the web server can read and write them.
```
sudo find $MOODLE_DATA_FOLDER/moodledata -type f -exec chmod 600 {} \;

```

# Call the cron.php in the moodle admin directory to run every minute.
```
echo "* * * * * /usr/bin/php $MOODLE_CODE_FOLDER/admin/cli/cron.php >/dev/null" | sudo crontab -u www-data -

```


Create Database and User
Moodle needs a database and a user with full privileges

 # Create a random password for the user who will access the moodle database
```
MYSQL_MOODLEUSER_PASSWORD=$(openssl rand -base64 6)

```


# Create a new database named moodle with the utf8mb4 character set and utf8mb4_unicode_ci collation.
sudo mysql -e "CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
# Create a new MySQL user moodleuser with a password
sudo mysql -e "CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY '$MYSQL_MOODLEUSER_PASSWORD';"
# Grant privileges on the moodle database to the moodleuser to allow localhost access.
sudo mysql -e "GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER ON moodle.* TO 'moodleuser'@'localhost';"
# Reloads privilege tables
sudo mysql -e "FLUSH PRIVILEGES;"
Configure Moodle from the command line
The last step can be done using the browser but we have all the information needed to complete the installation

MOODLE_ADMIN_PASSWORD=$(openssl rand -base64 8)
sudo chmod -R 0777 $MOODLE_CODE_FOLDER
cd $MOODLE_PATH
sudo -u www-data /usr/bin/php $MOODLE_CODE_FOLDER/admin/cli/install.php --non-interactive --lang=en --wwwroot="$PROTOCOL$WEBSITE_ADDRESS" --dataroot=$MOODLE_DATA_FOLDER/moodledata --dbtype=mariadb --dbhost=localhost --dbname=moodle --dbuser=moodleuser --dbpass="$MYSQL_MOODLEUSER_PASSWORD" --fullname="Generic Moodle" --shortname="GM" --adminuser=admin --summary="" --adminpass="$MOODLE_ADMIN_PASSWORD" --adminemail=please@changeme.com --agree-license
echo "Moodle installation completed successfully. You can now log on to your new Moodle at $PROTOCOL$WEBSITE_ADDRESS as admin with $MOODLE_ADMIN_PASSWORD and complete your site registration"
echo "Remember to change the admin email, name and shortname using the browser in your new Moodle"

```
  sudo find $MOODLE_CODE_FOLDER -type d -exec chmod 755 {} \;
  sudo find $MOODLE_CODE_FOLDER -type f -exec chmod 644 {} \;

```
# nginx needs slash arguments set
sudo sed -i "/require_once(__DIR__ . '\/lib\/setup.php');/i \$CFG->slasharguments = false;" /var/www/html/moodle/config.php
Prepare Development Database for Production
A production database contains student submissions. Extra steps are needed to ensure student data is secure from unauthorized access and safeguard against data loss.

Check Permissions
Permissions are used by Linux to control read, write and execute for directories and folders

# Reset the permissions on /var/www/html/moodle directories to read, write and execute for the webserver, read and execute for group and others
sudo find /var/www/html/moodle -type d -exec chmod 755 {} \;
# Reset the permissions on /var/www/html/moodle files to read, write for the webserver, read only for group and other
sudo find /var/www/html/moodle -type f -exec chmod 644 {} \
Set a root password on the database
# Run the mariadb-secure-installation script to strengthen security by setting a root password, removing anonymous users, disabling remote root login, deleting the test database, and reloading privileges.
# Don't forget to write down the root database in a safe place.
sudo mariadb-secure-installation
Configure and enable a firewall
# Allow SSH (port 22) for remote access
sudo ufw allow 22/tcp
# Enable the UFW firewall with confirmation
sudo ufw --force enable
# Set default policy to deny all incoming connections
sudo ufw default deny incoming
# Set default policy to allow all outgoing connections
sudo ufw default allow outgoing
# Allow HTTP traffic on port 80
sudo ufw allow www 
# Allow full access for Apache (HTTP and HTTPS)
sudo ufw allow 'Apache Full'
Convert http to the encrypted https protocol
NOTE: A domain name is required to get https in this step. Obtain one if you haven't already done so.
Using HTTPS over HTTP on a Moodle site provides the critical security benefits of encrypted data transmission and compliance with privacy regulations and some features in Moodle may not function properly or may be blocked if the site is using a HTTP connection.

Apache version
# Install package and run the certbot program 
sudo apt-get install certbot python3-certbot-apache
# Call the certbot and answer the prompts
sudo certbot --apache
# Check the new configuration is correct
sudo apache2ctl configtest
# Restart Apache
sudo systemctl reload apache2
Nginx version
sudo apt-get update
# Install needed packages
sudo apt-get install certbot python3-certbot-nginx
# Run the certbot script
sudo certbot --nginx
# Test your new configuration
sudo nginx -t
# Reload the webserver to allow changes to take effect 
sudo systemctl reload nginx
You must update references after changing your protocol.

#Replace oldsitehost ( ie http://yourdomain.com) with newsitehost ( ie https://yourdomain.com)
cd /var/www/html/moodle
sudo php admin/tool/replace/cli/replace.php --search=//oldsitehost --replace=//newsitehost --shorten --non-interactive
SSH keys
An SSH key consists of a public and private key pair used to securely authenticate using the Secure Shell (SSH) protocol without needing a password. The key is much longer and more complex than typical passwords, making them extremely resistant to brute-force attacks and guessing. Only the public key is copied (id_rsa.pub); never share your private key.

For Windows Users:

Open Settings > Apps & Features > Optional Features. Look for "OpenSSH Client" in the list
If it's not installed, click "Add a feature," find "OpenSSH Client," and install it.
Linux Users

Open a terminal on your home machine (not your server)
# Generate the SSH key pair by running the key generation (without sudo so the keys are stored in ~/.ssh/)
ssh-keygen
# Press Enter to accept the default key save location (usually C:\Users\your_username\.ssh\id_rsa for Windows users, ~/.ssh\id_rsa for Linux)
# When prompted, optionally enter a secure passphrase for extra protection or press Enter to leave it empty.
# Copy  your public key to the remote server's ~/.ssh/authorized_keys file, permissions will be sett automatically
ssh-copy-id user@server_address
# Try to log in, It should not ask you for a password.
Remove SSH root access
For security, you should always log in as a user with elevated privileges (a sudo user). You should never login as root on a production server and you should remove root SSH access from your server.

# If you don't already have a sudo user, as root, create a new user 'user1' with home directory and prompt for password
# Enter and confirm a password for user1. User details are optional
adduser user1
# Add 'user1' to the sudo group for administrative privileges
usermod -aG sudo user1
Check you can ssh into your server as your new sudo user! Leave another terminal open until confirmed.

# Backup sshd_config before modifying
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
# Open the ssh config file
sudo nano /etc/ssh/sshd_config
# Change the line containing PermitRootLogin to PermitRootLogin no
# Restart SSH service to apply changes
sudo systemctl restart ssh
Keep Ubuntu up to date
Applying security patches can be automated.

sudo apt install unattended-upgrades 
# Edit or review /etc/apt/apt.conf.d/20auto-upgrades:
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
These lines default to "1" for daily updates Other values: "7" (weekly), "0" (disabled)

APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
# Edit the config file to customize what will be upgraded
# Uncomment as needed
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";


Check the log /var/log/unattended-upgrades/

SQL Backups
Daily automated date marked sql backups should be set up on a production Moodle. Dumps can be large and need to be deleted periodically.

# Create MySQL backup user and grant privileges to lock tables (better security than using root or moodleuser roles)
BACKUP_USER_PASSWORD=$(openssl rand -base64 6)
mysql <<EOF
CREATE USER 'backupuser'@'localhost' IDENTIFIED BY '${BACKUP_USER_PASSWORD}';
GRANT LOCK TABLES, SELECT ON moodle.* TO 'backupuser'@'localhost';
FLUSH PRIVILEGES;
EOF
# Create a configuration file for root with the backup user credentials (to avoid the password being visible in the SQL backup call)
cat > /root/.my.cnf <<EOF
[client]
user=backupuser
password=${BACKUP_USER_PASSWORD}
EOF
# Set the configuration file permissions to read and write for root only
chmod 600 /root/.my.cnf
# Setup backup directory with read, write and execute permissions for root only
mkdir -p /var/backups/moodle && chmod 700 /var/backups/moodle && chown root:root /var/backups/moodle
# Set up two daily cron jobs, one to dump the database and the other to remove dumps more than a set number of days old (hardcoded to 7)
(crontab -l 2>/dev/null; echo "0 2 * * * mysqldump --defaults-file=/root/.my.cnf moodle > /var/backups/moodle/moodle_backup_\$(date +\%F).sql") | crontab -
(crontab -l 2>/dev/null; echo "0 3 * * * find /var/backups/moodle -name \"moodle_backup_*.sql\" -type f -mtime +7 -delete") | crontab -
Setting up antivirus on the server
In order to have your Moodle server scan uploaded files for viruses, you can set up Antivirus plugins.

ClamAV® is available to be installed in Ubuntu. Run the following command in Ubuntu to install the antivirus server:

# sudo apt install -y clamav
This installs the clam scanner for scanning files one-by-one. To activate it, go to Site Administration > Plugins > Antivirus plugins > Manage antivirus plugins > ClamAV antivirus and 'enable' the plugin by opening the eye.


Add this command to the 'Settings' for ClamAV®:

/usr/bin/clamscan
Note: This scanner launches the ClamAV® process once for each file uploaded. If you have a lot of files being uploaded simultaneously on your Moodle site, you may wish to activate the ClamAV® daemon process. To do that, install the ClamAV® daemon:

# sudo apt install -y clamav-daemon
Once the daemon process is running on the site, you can change the ClamAV® command in the Moodle settings to:

/usr/bin/clamdscan
In previous Moodle branches: Check "Use ClamAV on uploaded files" ClamAV Path : /usr/bin/clamscan Quarantine Directory : /var/quarantine

Save Changes

