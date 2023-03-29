# Deploy Laravel on AWS

Here is the steps to deploy a Laravel app on AWS EC2 instance:

Stack:
- Ubuntu 20.04 LTS
- Nginx
- PHP 8.2 FPM

## Login to EC2 instance:
```
$ chmod 600 pemfilename.pem
$ ssh -i “pemfilename.pem” username@ipaddressofinstance
```

## Install Nginx
```
sudo apt-get update
add-apt-repository ppa:nginx/$nginx
sudo apt install nginx
nginx -V
```

## Install and Configure MySQL
There are two ways to install a MySQL database.

* Using AWS EC2 as Localhost
* Using AWS RDS DB Instance

### Using AWS EC2 as Localhost
- Install MySQL in Ubuntu by running the below command.
```
sudo apt-get update
sudo apt install mysql-server
sudo mysql_secure_installation
sudo mysql
```

Once we are done with the installation process use the below command to configure the root account with an authentication password.
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'very_your_password';
FLUSH PRIVILEGES;
```

### Using AWS RDS DB Instance
For creating a MySQL database instance, go to the link below, there is a step-by-step guide: Create a MySQL DB instance and connect to a database

Since we are done with configuring our database let’s move towards the Laravel project setup. If you are familiar with the initial setup of laravel then feel free to skip the next section and jump to the deployment section.

## Set Up Laravel project

- Install PHP 8.2

```
sudo apt update
sudo apt upgrade
sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https
LC_ALL=C.UTF-8 sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.2
sudo apt install php8.2-cli php8.2-fpm php8.2-zip php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl
```

- Type `php -v` to check if PHP was installed successful.

```
Output:

PHP 8.2.1 (cli) (built: Jan 13 2023 10:43:08) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.2.1, Copyright (c) Zend Technologies
    with Zend OPcache v8.2.1, Copyright (c), by Zend Technologies
```

- Install Composer globally
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```
- Type `composer` to check if Composer was installed successful.

## Prepare your Laravel App and Deploy

### Generate ssh key pair to your application
```
$ ssh-keygen
```
- Now visit your GitHub account and on repository of your app, go to `Settings > Deploy Keys > New Key` and copy and paste you public key SSH (id_rsa.pub).

### Download your application from Git
```
$ git clone git@github.com/username:repository_app_name.git html
```

### Prepare application to build
```
$ cd html/
~/html$ composer install --optimize-autoloader --no-dev
~/html$ php artisan key:generate
~/html$ php artisan config:cache
~/html$ php artisan route:cache
~/html$ php artisan view:cache
~/html$ npm install && npm run build
~/html$ cp .env.example .env
~/html$ chmod -R 777 bootstrap
~/html$ chmod -R 777 storage
```
## Configure Virtual Host on Nginx
```
sudo nano /etc/nginx/sites-available/default
```

- Put this script below on `default` file
```
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    root /var/www/html/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

- Save the file and run the following commands to verify the Nginx syntax and restart Nginx for loading your changes.
```
sudo nginx -t
sudo service nginx restart 
```
- Delete `html` folder from `/var/www/`
```
/var/www$ sudo rm -rf html
```
- Now move your Laravel app that is in `html` folder to `/var/www`.
```
$ mv html/ /var/www/
```

## Visit http://EC2_PUBLIC_IPADDRESS
