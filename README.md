#Install WordPress
mkdir =p /var/www/mylab
cd /var/www/mylab
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm latest.tar.gz
chown -R www-data:www-data /var/www/mylab #Change the owner & group to www-data
 
/var/www/mylab/wordpress# cp wp-config-sample.php wp-config.php
  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define('DB_NAME', 'wordpress');
 
  /** MySQL database username */
  define('DB_USER', 'wordpressuser');
 
  /** MySQL database password */
  define('DB_PASSWORD', 'P@ssw0rd');
 
#Create Nginx Virtual Host File
nano /etc/nginx/sites-available/mylab
#Paste the following to /etc/nginx/sites-available/wordpress
  server {
      server_name dev.mylab.com;
      listen 80;
      port_in_redirect off;
      access_log /var/log/nginx/mylab.access.log;
      error_log /var/log/nginx/mylab.error.log;
 
      root /var/www/mylab/wordpress;
      index index.php;
 
      location / {
              try_files $uri $uri/ /index.php?$args;
      }
 
      access_log off;
      
      # Deny public access to wp-config.php
      location ~* wp-config.php {
      deny all;
      }
 
      location ~ .php$ {
              try_files $uri =404;
              include fastcgi_params;
              fastcgi_pass unix:/run/php/php7.0-fpm.sock;
              fastcgi_split_path_info ^(.+.php)(.*)$;
              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      }
 
    }  
 
#Unlink and link
unlink /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/mylab /etc/nginx/sites-enabled/mylab
 
#Increase the Upload File Size in Nginx
vi /etc/nginx/nginx.conf
#set client body size to 10MB
  http {
      ##
      # Basic Settings
      ##
      client_max_body_size 10M;
}
 
#Increase the MAX upload file size
nano /etc/php/7.0/fpm/php.ini
  upload_max_filesize = 10M
 
#Restart Services
service nginx restart
service php7.0-fpm restart
 
#login to Wordpress using hostname / IP Address
