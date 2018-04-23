# smallmelo.com migration part 1

## copy public ssh key to wisko
get public ssh key from local machine
```
cat ~/.ssh/id_rsa.pub
```
copy that log into wisko in seperate termibal window
```
nano .ssh/authorized_keys
```
paste in then exit

## changing the folder structure of my wordpress site

used [this site](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts) to setup the folders the way i wanted them. 

lots of nano-ing and reviewing later, ready to start making changes.

```
cd /var/www
```
make directory
```
sudo mkdir -p /var/www/allthehatsformaps.com/public_html
```

grant myself permission to access
```
sudo chown -R $USER:$USER /var/www/allthehatsformaps.com/public_html

```

create example page

```
nano /var/www/allthehatsformaps.com/public_html/index.html

```

ugh, do some renaming
```
sudo mv allthehatsformaps.com allthehatsformaps
sudo chown -R $USER:$USER /var/www/allthehatsformaps/public_html

```
add my html content accordingly, then make a copy of the conf file
```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/allthehatsformaps.conf

nano allthehatsformaps.conf

```
grant permissions
```
sudo chown -R $USER:$USER /etc/apache2/sites-available/allthehatsformaps.conf

```

edited my new config file
```
ServerName allthehatsformaps.com
ServerAlias www.allthehatsformaps.com
ServerAdmin melo@allthehatsformaps.com
DocumentRoot /var/www/allthehatsformaps/public_html

```

disable charles' site & enable my new one

```
sudo a2dissite 000-default.conf
sudo a2ensite allthehatsformaps.conf
service apache2 reload

```

copy wordpress folder from htdocs to my new folder 
```
sudo mv /var/www/htdocs/wordpress /var/www/allthehatsformaps/public_html

```

oops... that moved the entire folder, i just wanted to move its contents. 

```
cd /var/www/allthehatsformaps/public_html
sudo mv wordpress/* .
rmdir wordpress
rm index.html
```

going to allthehatsformaps.com works

going to allthehatsformaps.com/wp-admin tries to redirect to http://159.65.75.41/wordpress/wp-login.php?redirect_to=http%3A%2F%2Fallthehatsformaps.com%2Fwp-admin%2F&reauth=1 and throws a 404 error

went instead to:
http://159.65.75.41/wp-login.php

copy back the wordpress folder into htdocs

```
sudo cp /var/www/allthehatsformaps/public_html/* /var/www/htdocs/wordpress
```

disable my site config and enable old one

```
sudo a2dissite allthehatsformaps.conf
sudo a2ensite 000-default.conf
service apache2 reload
```

went to the wordpress login and signed in
under settings removed "wordpress" subfolder from the site url

disable charles' site & enable my new one

```
sudo a2dissite 000-default.conf
sudo a2ensite allthehatsformaps.conf
service apache2 reload

```

logged back into wordpress and changed the url to: http://allthehatsformaps.com

removed the old wordpress folder in htdocs:
```
sudo rm -r wordpress

```

cd back into the new wordpress and added a .htaccess file with the following:

```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>

```

then gave wordpress permission to edit it, according [to this site](https://stackoverflow.com/questions/23388144/wordpress-apache-permalinks-not-working-404-error)


