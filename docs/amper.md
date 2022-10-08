# Amper

> ## Getting Started

Amper is a software to make web hosting easier. It is required to have root access to your server.<br>
This software is free to use. If you experience any bugs, please join my discord in order to report it: https://discord.gg/BABBKBY7cg 

### Dependencies
* PHP 7.4, 8.0 or 8.1 with `cli`, `mysql`, `openssl`, `PDO`, `tokenizer`, `curl`, `zip`, and `fpm` for Nginx.
* MySQL 5.7 or higher, or MariaDB 10.2 or higher. MySQL 8 recommended.
* `nginx` (Apache is not supported yet)
* `composer`
* `curl`
* `unzip`
* `tar`

#### Install Dependencies

The commands to install the dependencies are below:

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
# Set up database
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash

# Update the repositories
apt update

apt -y install php8.1 php8.1-{fpm,curl,mysql,openssl,pdo,tokenizer,curl,zip} mariadb-server nginx tar unzip git
</pre>

#### Installing Composer

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
</pre>

### Downloading Files

Now we need to install all the files for the software.<br>
First we need to create a folder.

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
mkdir -p /var/www/amper && cd /var/www/amper
</pre>

Now we have created a folder for the files.<br>
All we need to do it download the files.

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
curl -Lo amper.tar.gz https://github.com/grcq/Amper/archive/refs/tags/v1.0.0.tar.gz
tar -xzvf amper.tar.gz

# To allow the websites to be uploaded.
chmod -R 755 /var/www /etc/nginx/sites-enabled
</pre>

### Database Setup

To set up the database, we need to access MySQL first by doing `mysql -u root -p`.<br>
Now do the following:

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
CREATE USER 'amper'@'127.0.0.1' IDENTIFIED BY 'passwordHere';
CREATE DATABASE amper;
GRANT ALL PRIVILEGES ON amper.* TO 'amper'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
</pre>

### Installation

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
componser install --no-dev --optimize-autoloader

php bin\console db:setup
php bin\console user:create
</pre>


> ## Webserver Configuration

Nginx with SSL.
First, remove the default NGINX configuration.

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
rm /etc/nginx/sites-enabled/default
</pre>

Now, you need to paste the contents of the file below and replace `<domain>` with your domain.<br>
Set the file name to `amper.conf` and place it in `/etc/nginx/sites-available`, or if you are on CentOS: `/etc/nginx/conf.d/`.

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
server_tokens off;

server {
    listen 80;
    server_name [domain];
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name [domain];

    root /var/www/amper/public;
    index index.php;

    access_log /var/log/nginx/amper-access.log;
    error_log  /var/log/nginx/amper-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    # SSL Configuration - Replace the example [domain] with your domain
    ssl_certificate /etc/letsencrypt/live/[domain]/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/[domain]/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

    # See https://hstspreload.org/ before uncommenting the line below.
    # add_header Strict-Transport-Security "max-age=15768000; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
</pre>

### Enabling the Configuration

Final step of your Nginx configuration:

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
# You do not need to symlink this file if you are using CentOS.
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf

# You need to restart nginx regardless of OS.
sudo systemctl restart nginx
</pre>

## Creating SSL Certificates

If you use Certbot:

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
sudo apt update
sudo apt install -y certbot
sudo apt install -y python3-certbot-nginx
</pre>

### Creating a Certificate

To create the certificate, please do the following:

<pre style="background-color: #000000; color: white; border-radius: 10px; padding: 20px;">
systemctl stop nginx

# Nginx
certbot certonly --nginx -d [domain]

# If Nginx does not work
certbot certonly --standalone -d [domain]

systemctl start nginx
</pre>