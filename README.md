# LEMP-server
* `sudo apt-get update`
* `sudo apt-get upgrade`
* `sudo apt-get install nginx`
* `sudo apt-get install mysql-server`
* `sudo mysql_secure_installation`
* `sudo add-apt-repository universe`
* `sudo apt-get install php-fpm php-mysql php-mbstring php-xml php-soap`

php fpm config

* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  cgi.fix_pathinfo=0`
* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  memory_limit = 32M`
* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  upload_max_filesize = 2M`
* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  post_max_size = 3M`
* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  max_execution_time = 300`
* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  max_input_time = 300`
* `sudo nano /etc/php/7.2/fpm/php.ini  -------->  max_file_uploads = 100`
`sudo systemctl restart php7.2-fpm`

nginx

* `sudo nano /etc/nginx/sites-available/default  -------->  client_max_body_size 2M;`
* `sudo nano /etc/nginx/sites-available/default`

nginx config

#server for redirecting from IP to DNS: ---->

`        server {
            listen 80;
            server_name 10.10.10.10;

            return 301 $scheme://hsshohada.com;
        }`

#default server config

`
server {
        client_max_body_size 100M;
        listen 80 default_server;
        listen [::]:80 default_server;
        
        #phpmyadmin server: ---->
        location /phpmyadmin {
                root /var/www/laravel/;
                index index.php index.html index.htm;
                location ~ ^/phpmyadmin/(.+\.php)$ {
                        try_files $uri =404;
                        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
                        include fastcgi_params;
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                }
                location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
                        root /var/www/laravel/;
                }
        }
        
        #laravel server:---->
        root /var/www/laravel/public;
        index index.php index.html index.htm;
        server_name hsshohada.com;
        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }
        location ~ \.php$ {
               include snippets/fastcgi-php.conf;
               fastcgi_pass unix:/run/php/php7.2-fpm.sock;
        }

        location ~ /\.ht {
               deny all;
        }
}
`
* `sudo nginx -t`
* `sudo systemctl reload nginx`
* `sudo mkdir -p /var/www/laravel`
* `sudo fallocate -l 1G /swapfile`
* `sudo mkswap /swapfile`
* `sudo swapon /swapfile`
* `cd ~`
* `curl -sS https://getcomposer.org/installer | php`
* `sudo mv composer.phar /usr/local/bin/composer`
* `cd /var`
* `sudo mkdir repo && cd repo`
* `sudo mkdir site.git && cd site.git`
* `sudo git init --bare`
* `sudo nano /var/repo/site.git/hooks/post-receive`

------->#!/bin/sh

------->git --work-tree=/var/www/laravel --git-dir=/var/repo/site.git checkout -f

* `sudo chmod +x post-receive`
* `composer install --no-dev`
* `sudo apt-get install phpmyadmin`
* `sudo chown -R :www-data /var/www/laravel`
* `sudo chmod -R 775 /var/www/laravel/storage`
* `sudo chmod -R 775 /var/www/laravel/bootstrap/cache`
* `mysql>`
* `CREATE DATABASE blog;`
* `SHOW DATABASES;`
* `exit`
* `sudo mysql -u root`
* `CREATE USER 'erfan'@'localhost' IDENTIFIED BY 'password';`
* GRANT ALL PRIVILEGES ON *.* TO 'erfan'@'localhost';`
* `FLUSH PRIVILEGES;`
* `exit`
* `sudo service mysql restart`
* `cp .env.example .env`
