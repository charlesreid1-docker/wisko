# smallmelo.com migration part 3 (allthehatsformaps to smallmelo)



## remove placeholder file

```
sudo rm index.html
```

## copy the files to smallmelo
```
sudo cp -a /var/www/allthehatsformaps/public_html/* /var/www/smallmelo/public_html
```

## update the site in wordpress

logged back into wordpress and changed the url to: [http://smallmelo.com](http://smallmelo.com/)


## copy the relevent stuff over to the smallmelo.conf

```
<Directory /var/www/smallmelo/public_html/>

  DirectoryIndex index.php

  AllowOverride All

  Order allow,deny

  Allow from all

  </Directory>

```

## disable and enable sites
```
sudo a2dissite allthehatsformaps.conf
sudo a2ensite smallmelo.conf
sudo /etc/init.d/apache2 restart

```

## give permissions

```
sudo chown -v :www-data /var/www/smallmelo/public_html/.htaccess

sudo chmod -v 664 "/var/www/smallmelo/public_html/.htaccess"


```
## reboot

```
sudo /etc/init.d/apache2 restart

```

## disabled, reenabled, reload

```
sudo a2dissite smallmelo.conf
sudo a2ensite smallmelo.conf
sudo /etc/init.d/apache2 restart
service apache2 reload

```

## disabling allthehats
```
sudo rm allthehatsformaps.conf
service apache2 reload
```

## in the /var/wwww
```
sudo rm -r allthehatsformaps

```

## in dreamhost 
remove wisko DNS records
setup redirect
