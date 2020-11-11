# phpIPAM-on-Ubuntu-20.04
Install and Configure phpIPAM on Ubuntu 20.04

Step 1: Install MariaDB  Server

  # sudo apt install software-properties-common mariadb-server mariadb-client
    systemctl status mariadb
  # sudo mysql_secure_installation 
      Set root password? [Y/n] y
      Remove anonymous users? [Y/n] y
      Disallow root login remotely? [Y/n] y
      Remove test database and access to it? [Y/n] y
      Reload privilege tables now? [Y/n] y
# mysql -V

# sudo mysql -u root -p

     CREATE DATABASE phpipam;
     GRANT ALL ON phpipam.* TO phpipam@localhost IDENTIFIED BY 'StrongDBPassword';
     FLUSH PRIVILEGES;
     QUIT;
     
Step 2: Install PHP and required modules

     sudo apt update 
     sudo apt -y install php php-{mysql,curl,gd,intl,pear,imap,memcache,pspell,tidy,xmlrpc,mbstring,gmp,json,xml,fpm}
     
Step 3: Install phpIPAM
     
     sudo git clone --recursive https://github.com/phpipam/phpipam.git /var/www/html/phpipam
     cd /var/www/html/phpipam
     
 Step 4: Configure phpIPAM on Ubuntu 20.04
 
     cd /var/www/html/phpipam
     sudo cp config.dist.php config.php
     
sudo nano config.php
/**
* database connection details
******************************/
$db['host'] = 'localhost';
$db['user'] = 'phpipam';
$db['pass'] = 'StrongDBPassword';
$db['name'] = 'phpipam';
$db['port'] = 3306;

Using Nginx Web server

    sudo systemctl stop apache2 && sudo systemctl disable apache2
    sudo apt -y install nginx
    
Configure nginx:
    sudo nano /etc/nginx/conf.d/phpipam.conf
    
    Ubuntu 20.04:
    
    server {
    # root directory
    root   /var/www/html;

    # phpipam
    location /phpipam/ {
        try_files $uri $uri/ /phpipam/index.php;
        index index.php;
    }
    # phpipam - api
    location /phpipam/api/ {
        try_files $uri $uri/ /phpipam/api/index.php;
    }

    # php-fpm
    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
        fastcgi_index  index.php;
        try_files      $uri $uri/ index.php = 404;
        include        fastcgi_params;
    }
 }

    sudo chown -R www-data:www-data /var/www/html
    sudo systemctl restart nginx
    
    sudo systemctl stop nginx && sudo systemctl disable nginx

    sudo apt -y install apache2
    sudo a2enmod rewrite
    sudo systemctl restart apache2
    
    sudo apt -y install libapache2-mod-php php-curl php-xmlrpc php-intl php-gd
    
    sudo nano /etc/apache2/sites-available/000-default.conf
    <VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot "/var/www/html/phpipam"
    ServerName phpipam.computingforgeeks.com
    ServerAlias www.phpipam.computingforgeeks.com
    <Directory "/var/www/html/phpipam">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "/var/log/apache2/phpipam-error_log"
    CustomLog "/var/log/apache2/phpipam-access_log" combined
</VirtualHost>

sudo systemctl restart apache2

sudo mysql -u root -p phpipam < /var/www/html/phpipam/db/SCHEMA.sql

Username: admin
Password: ipamadmin
