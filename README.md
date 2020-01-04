# LEMP-server
* `sudo apt-get update`
* `sudo apt-get upgrade`
* `sudo apt-get install nginx`
* `sudo apt-get install mysql-server`
* `sudo mysql_secure_installation`
* `sudo add-apt-repository universe`
* `apt-get install software-properties-common`
* `add-apt-repository ppa:ondrej/php`
* `apt-get update`
* `sudo apt-get install php-fpm php-mysql php-mbstring php-xml php-soap php-gd php-curl`
* `sudo apt-get install phpmyadmin`
* `sudo ln -s /usr/share/phpmyadmin/ /var/www/laravel/`
* `sudo apt-get install composer`
* `composer global require hirak/prestissimo`

php fpm config

* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  cgi.fix_pathinfo=0`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  memory_limit = 32M`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  upload_max_filesize = 2M`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  post_max_size = 3M`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  max_execution_time = 300`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  max_input_time = 300`
* `sudo nano /etc/php/7.4/fpm/php.ini  -------->  max_file_uploads = 100`

`sudo systemctl restart php7.4-fpm`

certbot

* `sudo add-apt-repository ppa:certbot/certbot`
* `sudo apt-get update`
* `sudo apt-get install certbot python-certbot-nginx`
* `sudo certbot certonly --nginx`
* `sudo certbot renew --dry-run`


nginx

* `sudo nano /etc/nginx/sites-available/default`

nginx config
```
# In the name of Allah
server {
        listen [::]:443 ssl ipv6only=on;
        listen 443 ssl http2;
        ssl_certificate /etc/letsencrypt/live/hsshohada.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/hsshohada.com/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        # SSL Pre-Config
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        ssl_stapling on;
        ssl_stapling_verify on;
        ssl_trusted_certificate /etc/letsencrypt/live/hsshohada.com/fullchain.pem;
        resolver 1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 valid=3600s;
        resolver_timeout 5s;

        client_max_body_size 100M;

        #phpmyadmin server: ---->
        location /phpmyadmin {
                root /var/www/laravel/;
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
        #laravel server:---->
        root /var/www/laravel/public;
        index index.php index.html index.htm;
        server_name hsshohada.com;
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

}
server {
        listen 80;
        server_name hsshohada.com;
        error_log   /dev/null   crit;
        access_log off;
        location / {
                return 301 https://$host$request_uri;
        }

}
#server for redirecting from IP to DNS: ---->
server {
        listen 80;
        server_name 145.239.165.142;
        return 301 $scheme://hsshohada.com;
}
```
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
```(paste lines below:)
#!/bin/sh

echo "*******\nPost receive hook: Updating website\n*******"
git --work-tree=/var/www/laravel --git-dir=/var/repo/site.git checkout -f

cd /var/www/laravel

echo "*******\ncomposer install\n*******"
composer install  >> /dev/null 2>&1

echo "*******\nmigrating\n*******"
php artisan migrate --no-interaction --force

echo "*******\nhandling cache\n*******"
php artisan cache:clear
php artisan config:cache
php artisan route:cache
php artisan view:clear
php artisan view:cache

echo "*******\nALL HAIL ERFAN\n*******"
```
* `sudo chmod +x post-receive`

after git push to server with: `git remote add production ssh://root@10.10.10.10/var/repo/site.git`

* `composer install --no-dev`
* `sudo chown -R :www-data /var/www/laravel`
* `sudo chmod -R 775 /var/www/laravel/storage`
* `sudo chmod -R 775 /var/www/laravel/bootstrap/cache`
* `sudo chmod -R 777 /var/www/laravel/temp`
* `cp .env.example .env`
