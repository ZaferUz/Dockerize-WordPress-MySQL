# 1. Run WordPress with Docker Compose

- At this moment we need to do three things:

1. Build the image
2. Run MySQL
3. Run the image

- You can simplify these steps using Docker Compose, a tool for defining and running multi-container Docker applications.

- Start with adding *docker-compose.yml* to the Dockerfile that you defined in the previous steps with the following content:

```
version: "3.7"

services:
  db:
    image: mysql:latest
    container_name: bfe-demo-db
    restart: always
    volumes:
      - db_data:/var/lib/mysql
      - ./backups:/backups
    environment:
      MYSQL_DATABASE: bfedemo
      MYSQL_USER: bfedemo_user
      MYSQL_PASSWORD: bfedemo_007
      MYSQL_ALLOW_EMPTY_PASSWORD: "no"
      MYSQL_ROOT_PASSWORD: bfedemo_root_007
  wordpress:
    image: bfe-demo-wp-php
    container_name: bfedemo_wordpress
    restart: always
    depends_on: ["db"]
    ports: ["80:80"]
    links: ["db:db"]
#    volumes:
#      - "./wp-data:/var/www/html"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: bfedemo_wordpress
      WORDPRESS_DB_USER: bfedemo_wordpress_user
      WORDPRESS_DB_PASSWORD: bfedemo_007
  phpmyadmin:
    image: phpmyadmin/phpmyadmin:4.7
    container_name: bfe_php_admin
    restart: always
    depends_on: ["db"]
    ports: ["12000:80"]
    links: ["db:db"]
    environment:
      - PHP_HOST=db
      - PHP_USER=bfedemo_wordpress_user
      - PHP_PASSWORD=bfedemo_007

volumes:
  db_data: {}
```

- In this file we define two main services (and assistant service) that are automatically linked with each other.
- Data in the containers is **not permanent**. This means that if you stop the container and run it again, there will no longer be any data inside. It's not very convenient as you have to initiate your WordPress every time. So we added *volumes* for db server to solve the problem. We do not need any volmes for wp server. Because we use our base image, built/created with its own data.

    
    - **db** – here we run the MySQL image from the Docker Hub in version *latest* with the qwerty password passed via an environment variable (the MySQL image is designed to handle such variables, too)
    - **wordpress** – here we build a Docker image based on the Dockerfile created before and map port 80 on the host to port 80 inside the container. The image, used here, is based on *wordpress:php7.4*. Then we pass the MySQL password via an environment variable (the WP base image is designed to handle such variables)
    - **phpadmin** - phpMyAdmin is a free software tool written in PHP, intended to handle the administration of MySQL over the Web. phpMyAdmin supports a wide range of operations on MySQL and MariaDB. Frequently used operations (managing databases, tables, columns, relations, indexes, users, permissions, etc) can be performed via the user interface, while you still have the ability to directly execute any SQL statement.
    - **volumes** - Data in the containers.

- Right now, we can run Docker Compose and start both containers with one command:
```
$ docker-compose up -d
```
Once the containers have started, you can open the URL in the web-browser and start using your application:
```
http://127.0.0.1:80
```

To stop the application, run:
```
$ docker-compose down
```
If you need to rebuild the WordPress image (eg. because you changed something in the sources), run:
```
$ docker-compose up -d --build
```

- **Docker will create a volume for you in the /var/lib/mysql folder. This volume will persist as long as you don't type ```docker-compose down -v```**.

# 2. Enabling SSL connection from WordPress to DataBase.

  
- to enable SSL connection from WordPress to db we can use plugins. 
- for manuel set up follow the steps below:
    1. So lets open ```wp-admin``` dashboard to install plugins. 
    2. click on the ```plugins``` on the Dashboard and click ```Add Plugins``` to add a new plugin on the opening page. 
    3. to install ssl plugin write ```Realy Simple SSL``` to search box
    4. click ```install``` and than click ```Active``` button.  
    5. finally click ```Go ahead, activate SSL!``` to start SSL connection. 
  

- to enable SSL connection from WordPress to db we can configure our dockerfile to use SSL plugins. Follow the steps below:
    1. to install SSL plugins open the link https://wordpress.org/plugins/really-simple-ssl/ and download the Realy Simple SSL Plugin folder. 
    2. unzip and copy the folder to /docker/mount_folder/plugins/
    3. change the dockerfile to enable COPY the file to image:

```
FROM wordpress:php7.4
COPY mount_folder/themes/neve /var/www/html/wp-content/themes/neve/
#COPY mount_folder/themes/kadence /var/www/html/wp-content/themes/kadence/
COPY mount_folder/plugins/preferred-languages /var/www/html/wp-content/plugins/prefered-languages/
COPY mount_folder/plugins/realy-simple-ssl /var/www/html/wp-content/plugins/realy-simple-ssl/
```
  4.  Build and use the image to enable SSL connection from WordPress to db. 


### 3. Installing and enabling Xdebug

- *https://xdebug.org/docs/install*

# This section describes on how to install Xdebug.

- Installing Xdebug with a package manager is often the fastest way. Depending on your distribution, run the following command:

- *Note:* You compile Xdebug separately from the rest of PHP. You need access to the scripts phpize and php-config. If your system does not have phpize and php-config, you will need to install the PHP development headers.

- Debian users can do that with:
```
apt-get install php-dev
```
- And RedHat and Fedora users with:
```
yum install php-devel
```

- It is important that the source version matches the installed version as there are slight, but important, differences between PHP versions. Once you have access to phpize and php-config, take the following steps:

1. Run ```php -i``` and copy the output of the following command and paste it on``` https://xdebug.org/wizard.php```. Follow the instructions there to install xDebug.

2. Clone the source of the latest/right xdebug version : ```git clone git://github.com/xdebug/xdebug.git```
   
3. or download ```https://xdebug.org/download#releases```

4. Unpack the downloaded file with ```tar -xvzf xdebug-3.0.3.tgz```

5. Run: ```cd xdebug-3.0.3```

6. Run: ```phpize```
   
   *Note: If phpize is not in your path, please make sure that it is by expanding the PATH environment variable. Make sure you use the phpize that belongs to the PHP version that you want to use Xdebug with.*

7. Run: ```./configure --enable-xdebug```

8. Run: ```make``` 
9.  Run: ```make install``` 

10. First check the ```xdebug.so``` file in ```/usr/lib/php/20190902``` (or find the right path )
 
    *Note: if not exist fint and copy it under the path, for example: ```cp modules/xdebug.so /usr/lib/php/20190902```*

10. Edit ```/etc/php/7.2/cli/php.ini``` and add the line ```zend_extension = /usr/lib/php/20190902/xdebug.so```

# Config Xdebug

1. Add the following line to php.ini:
```
zend_extension=/wherever/you/put/it/xdebug
```
- for example:
```
zend_extension = /usr/lib/php/20190902/xdebug.so
xdebug.remote_enable = 1
xdebug.remote_connect_back = 1
xdebug.remote_port = 9000
```
2. Restart your webserver, or PHP-FPM, depending on what you are using.

3. Verify that Xdebug is now loaded. 
4. Create a PHP page that calls **xdebug_info()** (https://xdebug.org/docs/all_functions#xdebug_info). If you request the page through the browser, it should show you an overview of Xdebug's settings and log messages.

5. On the command line, you can also run ```php -v```. Xdebug and its version number should be present as in:
```
PHP 7.4.10 (cli) (built: Aug 18 2020 09:37:14) ( NTS DEBUG )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
  with Zend OPcache v7.4.10-dev, Copyright (c), by Zend Technologies
  with Xdebug v3.0.0-dev, Copyright (c) 2002-2020, by Derick Rethans
```