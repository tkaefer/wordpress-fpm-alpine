[![Docker Repository on Quay](https://quay.io/repository/tkaefer/wordpress-php71-fpm-alpine/status "Docker Repository on Quay")](https://quay.io/repository/tkaefer/wordpress-php71-fpm-alpine)

## This image?

clone of official wordpress docker image

## Why?

Because Wordpress 4.9 wasn't available in a reasonable time.

## How?

`docker-compose.yaml`:
```
web:
  image: nginx:alpine
  restart: unless-stopped
  volumes:
    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
  links:
    - wordpress
  volumes_from:
    - wordpress
  ports:
    - "80:80"
wordpress:
  image: wordpress:php7.0-fpm-alpine
  restart: unless-stopped
  links:
    - mariadb:mysql
  volumes:
    - ./wordpress-data/wp-content:/var/www/html/wp-content
mariadb:
  image: mariadb
  restart: unless-stopped
  environment:
    - MYSQL_ROOT_PASSWORD=password
  volumes:
    - ./mariadb-data:/var/lib/mysql
```

`./nginx/nginx.conf`:
```
user nginx;

events {
  worker_connections 768;
}

http {
  upstream backend {
    server wordpress:9000;
  }

  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  gzip on;
  gzip_disable "msie6";

  server {
    listen 80;

    root /var/www/html/;
    index index.php index.html index.htm;

    location / {
      # try_files $uri $uri/ =404;
      try_files $uri $uri/ /index.php?$args;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
      root /usr/share/nginx/html;
    }

    location = /favicon.ico {
      log_not_found off;
      access_log off;
    }

    location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
      access_log off; log_not_found off; expires max;
    }


    location ~ [^/]\.php(/|$) {
      fastcgi_split_path_info ^(.+?\.php)(/.*)$;
      if (!-f $document_root$fastcgi_script_name) {
        return 404;
      }
      # This is a robust solution for path info security issue and works with "cgi.fix_pathinfo = 1" in /etc/php.ini (default)

      include fastcgi_params;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PHP_VALUE "upload_max_filesize=64m
      post_max_size=64m";
      fastcgi_pass wordpress:9000;
    }
  }
}
```
