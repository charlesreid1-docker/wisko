# wisko FAQ

## Apache

Link: [Apache on the Ubuntu Wiki](https://help.ubuntu.com/lts/serverguide/httpd.html)

Apache 2 was installed using the command `apt-get install apache2`.

### running apache2

To start, restart, or stop apache2 (it should start automatically on boot):

```
$ sudo service apache2 start
$ sudo service apache2 restart
$ sudo service apache2 stop
```

### apache2 configuration files

Apache keeps its configuration files in `/etc/apache2/`.

Apache is configured to host multiple websites, so there are multiple configuration files. 

The global Apache config file is `/etc/apache2/apache2.conf`.

Site-specific Apache config files are in `/etc/apache2/sites-available`. To make a new site, just add a new file to that folder (name it whatever you'd like). There is an existing default configuration in that folder.

The site configuration file is where you edit the folder it serves up, the port it listens on, what requests it is listening for (subdomain or full domain), etc.

When you have edited a site's configuration file, enable it using the `a2ensite` command:

```
$ a2ensite name_of_site
```

Likewise, disable the site with `a2dissite`:

```
$ a2dissite name_of_site
```

## PHP

Link: [LAMP Server on Ubuntu Wiki](https://help.ubuntu.com/community/ApacheMySQLPHP)

PHP was installed using aptitude.

### running php

PHP is a programming language designed for web browsers, so you don't need to start it - Apache will handle that. (Apache has a PHP module that has been enabled.)

### php configuration files

PHP also has multiple configuration files, but we're only interested in the Apache-PHP configuration file at `/etc/php5/apache2/php.ini`. The main thing that you would change here is the max file upload size (10 MB by default) and whether PHP prints out errors ("debug" mode).

## MySQL

Link: [MySQL on Ubuntu Wiki](https://help.ubuntu.com/lts/serverguide/mysql.html)

MySQL server was installed using aptitude.

### networking

MySQL will listen on port 3306 for incoming requests on the local machine *only* (it is bound to the address 127.0.0.1:3306). That means no outside computers can communicate with MySQL. This can be changed in the configuration file.

### running mysql

To run the MySQL server in the background (it should start automatically on boot), run:

```
$ sudo service mysql start
```

To get a MySQL shell, run MySQL from the command line:

```
$ mysql -u root -p
```

This will prompt you for your MySQL password.

To make a new MySQL user, log in as the root user and run:

```
mysql> CREATE USER 'newuser'@'localhost' IDENTIFIED BY "newpassword";
```

To create a database and give the new user permission to edit that database:

```
mysql> CREATE DATABASE newdb;
mysql> GRANT ALL PRIVILEGES ON newdb.* TO "newuser"@"localhost";
```

MySQL is picky about quotations.

### mysql configuration files

The main MySQL configuration file is in `/etc/mysql/mysql.cnf`.

The MySQL configuration file that you should edit is at `/etc/mysql/mysql.conf.d/mysql.cnf`.

## Gitea

### gitea locations

Gitea was installed using `go get`, which integrates Go with Github. The command checked out a copy of the code in the directory specified by `$GOCODE`, which is at `~/gocode`.

The gitea source is located in `/home/melo/gocode/src/code.gitea.io/gitea/`, which is symlinked to `~melo/gitea/`.

### running gitea

Gitea will *not* run on boot - a startup service needs to be added in `/etc/init.d/gitea` that will run the gitea command as user `melo`.

The Gitea web server is a binary Go program that bundles together everything it needs to run. The binary is built and run in the directory `~melo/gocode/src/code.gitea.io/gitea` (both as the user `melo`). 

To run the binary, you would execute it with the "web" argument:

```
$ cd ~/gocode/src/code.gitea.io/gitea/
$ ./gitea web
```

However, this starts the process in the foreground, so it's more convenient to run it in a screen (which allows you to detach from a shell, end your session, and reattach to that shell in a later session).

```
$ screen -d -m -S giteaweb $HOME/gocode/src/code.gitea.io/gitea/gitea web
```

This will start a new screen in detached mode (`-d -m`), call it `giteaweb` (`-S`), and run the command `gitea web`.

To reattach, list your screens using the `-ls` flag and reattach using the `-r` flag:

```
$ screen -ls
$ screen -r giteaweb
```

### rebuilding gitea

The gitea source directory at `~/gocode/src/code.gitea.io/gitea` is a cloned git repository, so you can check out different versions or branches as needed. If you make changes to the configuration file or template files, you will also need to re-make the gitea binary (since all of that is bundled up in the binary).

To remake the binary:

```
$ cd $GOCODE/src/code.gitea.io/gitea/

# here you can do stuff like
# git branch -a
# git checkout v1.3

$ TAGS="bindata" make generate build
```

