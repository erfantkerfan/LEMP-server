# LEMP-server
* `sudo apt-get update`
* `sudo apt-get upgrade`
* `sudo apt-get install nginx`
* `sudo apt-get install mysql-server`
* `sudo mysql_secure_installation`
* `sudo add-apt-repository universe`
* `sudo apt-get install software-properties-common`
* `sudo add-apt-repository ppa:ondrej/php`
* `sudo apt-get update`
* `sudo apt-get install php-fpm php-mysql php-mbstring php-xml php-soap php-gd php-curl php-imagick`
* `cd /tmp && sudo wget https://files.phpmyadmin.net/phpMyAdmin/5.0.4/phpMyAdmin-5.0.4-english.tar.gz`
* `sudo tar xvzf phpMyAdmin-5.0.4-english.tar.gz`
* `sudo mv phpMyAdmin-5.0.2-english /usr/share/phpmyadmin`
* `sudo sed -e "s|cfg\['blowfish_secret'\] = ''|cfg['blowfish_secret'] = '$(openssl rand -base64 32)'|" /usr/share/phpmyadmin/config.sample.inc.php > /usr/share/phpmyadmin/config.inc.php`
* `mkdir /usr/share/phpmyadmin/tmp`
* `chmod 777 /usr/share/phpmyadmin/tmp`
* `sudo mkdir -p /var/www/ && sudo ln -s /usr/share/phpmyadmin/ /var/www/`

php fpm config

* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  cgi.fix_pathinfo=0`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  memory_limit = 32M`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  upload_max_filesize = 2M`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  post_max_size = 3M`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  max_execution_time = 300`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  max_input_time = 300`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  max_file_uploads = 100`

* `sudo systemctl restart php7.4-fpm`

certbot

* `sudo add-apt-repository ppa:certbot/certbot`
* `sudo apt-get update`
* `sudo apt-get install certbot python3-certbot-nginx`

set-up YOUR-DOMAIN.COM
* `sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/YOUR-DOMAIN`
* `sudo ln -s /etc/nginx/sites-available/YOUR-DOMAIN /etc/nginx/sites-enabled/`
* `sudo rm /etc/nginx/sites-enabled/default`
* `sudo nano /etc/nginx/sites-available/YOUR-DOMAIN  -------->  server_name _`
* `sudo systemctl reload nginx`

certbot

* `sudo certbot certonly --nginx`
* `sudo certbot renew --dry-run`

nginx

* `sudo nano /etc/nginx/sites-available/YOUR-DOMAIN`

nginx config
```
server {
        listen 443 ssl http2;
        server_name YOUR-DOMAIN.COM;
        ssl_certificate /etc/letsencrypt/live/YOUR-DOMAIN.COM/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/YOUR-DOMAIN.COM/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        # SSL Pre-Config
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        ssl_stapling on;
        ssl_stapling_verify on;
        ssl_trusted_certificate /etc/letsencrypt/live/YOUR-DOMAIN.COM/fullchain.pem;
        resolver 1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 valid=3600s;
        resolver_timeout 5s;

        client_max_body_size 50M;

        # phpmyadmin server: ---->
        location /phpmyadmin {
                root /var/www/;
                index index.php index.html index.htm;
                location ~ ^/phpmyadmin/(.+\.php)$ {
                        try_files $uri =404;
                        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
                        include fastcgi_params;
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                }
                location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
                        root /var/www/laravel/;
                }
        }
        # laravel server:---->
        root /var/www/YOUR-DOMAIN/public;
        index index.php index.html index.htm;
        location / {
        try_files $uri $uri/ /index.php?$query_string;
        }
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        #       # With php7.0-cgi alone:
        #       fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        }
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        location ~ /\.ht {
                deny all;
        }
        #error_log   /dev/null   crit; # if you are confident
        #access_log off; # if you are confident
}
server {
        listen 80;
        server_name YOUR-DOMAIN.COM;
        error_log   /dev/null   crit;
        access_log off;
        location / {
                return 301 https://$host$request_uri;
        }
}
# server for redirecting from IP to DNS: ---->
server {
        listen 80;
        listen 443;
        error_log   /dev/null   crit;
        access_log off;
        server_name YOUR-IP;
        return 301 https://YOUR-DOMAIN.COM/$request_uri;
}
```
* `sudo nginx -t`
* `sudo systemctl reload nginx`

make a swapfil

* `sudo fallocate -l 1G /swapfile`
* `sudo mkswap /swapfile`
* `sudo swapon /swapfile`

isntall Composer

* `cd ~ && curl -sS https://getcomposer.org/installer | php`
* `sudo mv composer.phar /usr/local/bin/composer`
* `composer global require hirak/prestissimo`

prepare git server git-hook

* `cd /var && sudo mkdir repo && cd repo`
* `sudo mkdir YOUR-DOMAIN.git && cd YOUR-DOMAIN.git`
* `sudo git init --bare`
* `sudo nano hooks/post-receive`
```(paste lines below:)
#!/bin/sh

echo "*******\n Post receive hook activate: Updating website \n*******"
git --work-tree=/var/www/YOUR-DOMAIN --git-dir=/var/repo/YOUR-DOMAIN.git checkout -f

cd /var/www/YOUR-DOMAIN

echo "*******\n composer install \n*******"
composer install --no-dev >> /dev/null 2>&1

echo "*******\n migrating \n*******"
php artisan migrate --no-interaction --force

echo "*******\n handling cache \n*******"
php artisan cache:clear
php artisan config:cache
php artisan route:cache
php artisan view:clear
php artisan view:cache

echo "*******\n All Done! \n*******"
```
* `sudo chmod +x hooks/post-receive`

Prepare the server:

* `cd ~ && sudo mkdir -p /var/www/YOUR-DOMAIN && sudo chown -R :www-data /var/www/YOUR-DOMAIN`

git push to server on your local code-base with:

* `git init && git add . && git commit -m "initial commit"`
* `git remote add production ssh://USER@YOUR-IP/var/repo/YOUR-DOMAIN.git`
* `git push production master`

You get bunch of errors let fix it:
* `mysql -u root`
```
CREATE USER 'USER'@'localhost' IDENTIFIED BY 'PASSWORD';
CREATE DATABASE YOUR-DOMAIN CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
GRANT ALL ON YOUR-DOMAIN.* TO 'USER'@'localhost';
FLUSH PRIVILEGES;
```
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
* `cd /var/www/YOUR-DOMAIN && sudo cp .env.example .env`
* `sudo chown -R :www-data /var/www/YOUR-DOMAIN/`
* `sudo chmod -R 775 /var/www/YOUR-DOMAIN/storage`
* `sudo chmod -R 775 /var/www/YOUR-DOMAIN/bootstrap/cache`
* `sudo chmod -R 775 /var/www/YOUR-DOMAIN/public`
* `sudo chmod -R 777 /var/www/YOUR-DOMAIN/temp` --------> if needed
* `sudo nano .env`
