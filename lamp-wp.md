---
layout: default
---

# [BASH] Scripting installation of LAMP stack and WordPress

* * *

### I have written a little script, which automates the installation of LAMP and WordPress on Ubuntu 20.04.
You can find this script at my <a href="https://github.com/P4nd4233/automation" target="_blank">Git Hub Repository</a> for the automation articles.

> The Script takes the following parameters:
> * -m = MYSQL ROOT PASSWORD from the prompt when running `mysql_secure_installation`
> * -u = WP_USERNAME in MySQL DB
> * -p = WP_PASSWORD in MySQL DB
>
> This script definitely is not for production use, it is just a quick example of some bash coding. <br>
> Be aware that it overwrites some files. <br>
> The script creates a log for `mysql_secure_installation` at `/root/mysql-log.txt` .<br>
> You should run the script with root privileges, because it does package installation, configuration modifications, etc ...

## For example, you can run:

`sudo bash lamp-wordpress.sh -m SecUre123 -u wp_user -p wp_pass`

## The Code itself:


```bash
#!/bin/bash

echo_usage()
{
	echo "Usage: $0 -m <MYSQL ROOT PASSWORD> -u <WP_USERNAME> -p <WP_PASSWORD> "
	exit 1 
}

while getopts ":m:u:p:" arg; do
  case $arg in
    m) msp=$OPTARG;;
    u) wpu=$OPTARG;;
    p) wpp=$OPTARG;;
  esac
done

if [[ $msp = '' || $wpu = '' || $wpp = '' ]]; then
	echo_usage
fi

function run_script {
# Installing all necessary packages
apt install -y apache2 unzip expect curl mysql-server php libapache2-mod-php php-mysql php-cli php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip

# Enabling the services
systemctl enable --now apache2 mysql


#Runnig automated mysql_secure_installation
SECURE_MYSQL=$(expect -c "

set timeout 10
spawn mysql_secure_installation

expect \"Press y|Y for Yes, any other key for No:\"
send \"n\r\"

expect \"New password:\"
send \"$msp\r\"

expect \"Re-enter password:\"
send \"$msp\r\"

expect \"Remove anonymous users? (Press y|Y for Yes, any other key for No) :\"
send \"y\r\"

expect \"Disallow root login remotely? (Press y|Y for Yes, any other key for No) :\"
send \"y\r\"

expect \"Remove test database and access to it? (Press y|Y for Yes, any other key for No) :\"
send \"y\r\"

expect \"Reload privilege tables now? (Press y|Y for Yes, any other key for No) :\"
send \"y\r\"

expect eof
")

echo "$SECURE_MYSQL" >> ~/mysql-log.txt


#Creating Wordpress Database with credentials you have set

mysql -u root -p"$msp" -e "CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_520_ci;"
mysql -u root -p"$msp" -e "CREATE USER '$wpu'@'localhost' IDENTIFIED BY '$wpp';"
mysql -u root -p"$msp" -e "GRANT ALL ON wordpress.* TO '$wpu'@'localhost';"
mysql -u root -p"$msp" -e "FLUSH PRIVILEGES;"

#Creating temporary directory to download the latest WordPress in it
mkdir /tmp/wp

#Downloading WordPress
wget https://wordpress.org/latest.zip -O /tmp/wp/latest.zip

#Unzipping the archive at /var/www/wordpress
echo "Unzipping archive ..."
unzip /tmp/wp/latest.zip -d /var/www/ 1>/dev/null
if [[ "$?" -eq "0" ]]; then
	echo "Error Unzipping Archive ..."
else
	echo "Unzipped archive ..."
fi

#Changing Apache's config to allow Overrides
tee /etc/apache2/sites-available/wordpress <<EOF
<Directory /var/www/wordpress/>
	AllowOverride All
</Directory>	
EOF

touch /var/www/wordpress/.htaccess
mkdir /var/www/wordpress/wp-content/upgrade

a2enmod rewrite
#Fixing Permission issues
chown -R www-data:www-data /var/www/wordpress
find /var/www/wordpress/ -type d -exec chmod 750 {} \;
find /var/www/wordpress/ -type f -exec chmod 640 {} \;
sed -i '12s/html/wordpress/' /etc/apache2/sites-available/000-default.conf
systemctl restart apache2
}

# Calling the installation script from above

run_script
```