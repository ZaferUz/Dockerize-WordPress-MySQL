# Define a Dockerfile for WordPress 

- The first thing we need to do is to define how our image will look like in a Dockerfile. It is a text file that is added to the directory with the sources of your application.
- In general, our Dockerfile will consist of two commands:
  1. **FROM** – to build the application, use the official WP image **"FROM wordpress:php7.4"**
  2. **COPY** – copy my code to the defined directory in my image.
   
- Below you'll see how the Dockerfile will look depending on the contents of your repo: 
## WP Engine
  - If your repository contains the *entire engine together* with the contents, plugins and themes, make sure to add a Dockerfile with the following content along wp-admin, wp-content, and wp-includes directories:
  ```
    FROM wordpress:php7.4
    COPY . /var/www/html
  ```
## WP Theme

  - If your repository contains *only the sources of the theme* that you deploy, add a Dockerfile with the following content to the directory with your theme's sources:
```
FROM wordpress:php7.4
COPY . /var/www/html/wp-content/themes/mytheme/
```
## WP Plug-in

- If your repository contains *only the sources of the plug-ins* that you deploy, add a Dockerfile with the following content to the directory with your plug-in's sources:
```
FROM wordpress:php7.4
COPY . /var/www/html/wp-content/plugins/myplugin/
```
## WP Plug-in and theme
- If your repository contains plug-ins and themes, but lacks the WP Engine, you must use the Dockerfile to upload them to the proper location.

- For instance, let's assume that your repository contains 4 directories: *mytheme1, mytheme2, myplugin1* and *myplugin2*. 
  
- In this case, you have to add the Dockerfile along all these directories with the following content:

```
FROM wordpress:php7.4
COPY mount_folder/themes/neve /var/www/html/wp-content/themes/neve/
#COPY mount_folder/themes/kadence /var/www/html/wp-content/themes/kadence/
COPY mount_folder/plugins/preferred-languages /var/www/html/wp-content/plugins/prefered-languages/
#COPY mount_folder/plugins/realy-simple-ssl /var/www/html/wp-content/plugins/realy-simple-ssl/

```


## Build WP Docker Image
- Once you have your Dockerfile defined, you can build the image in the terminal:

```
$ cd docker
$ docker build -t bfe-demo-wp-php .
```
- The command will build an image named **bfe-demo-wp-php** in the context of your repository (the . at the end). The first execution will take a while since the official WordPress image needs to be downloaded to your disk. Next builds will be much faster.

Run WP Image
