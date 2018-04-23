# smallmelo.com migration part 2 (all the things)

## grant melo permission to create files and upload via sftp using filezilla

used [these](https://www.digitalocean.com/community/questions/i-can-t-seem-to-grant-myself-write-permissions-to-the-www-directory) instructions

## copied html/php files into var/www/smallmelo

using filazilla

## add the dns records to dreamhost to point to the server

159.65.75.41 (a)
www smallmelo.com (cname)


## create new conf file for smallmelo

```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/smallmelo.conf
```

## edit the conf file to point to smallmelo.com

## enable the site

```
sudo a2ensite smallmelo.conf
service apache2 reload
```
## make a backup copy all files from smallmelo.com 

## made a basic index.html file

## turn off hosting by dreamhost

## restart apache2
```
sudo service apache2 restart
```


