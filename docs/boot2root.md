# wisko

## Getting to a Shell

### Hardware

wisko is a higher-end 1-CPU Digital Ocean droplet.
* 1 CPU
* 50 GB SSD
* 2 GB RAM

IP address: 159.65.75.41

### Spin Up Hardware

* Follow the Digital Ocean steps to spin up a droplet in the SF region.

* Once the droplet is created, get the IP address.

* Pro tip: find your ssh key in `~/.ssh/id_rsa.pub` and paste it in during the Droplet creation process to avoid password issues.

### Set DNS Records

Add the following DNS records:

**A Record**:
* Record: none (set to @ automatically)
* Value: 138.68.10.168

**A Record**:
* Record: git
* Value: 138.68.10.168

**CNAME Record**:
* Record: 

### Connect to Droplet

Check that the Droplet has been booted and is listening:

```plain
$ ping -c 4 138.68.10.168
PING 138.68.10.168 (138.68.10.168): 56 data bytes
64 bytes from 138.68.10.168: icmp_seq=0 ttl=56 time=28.494 ms
64 bytes from 138.68.10.168: icmp_seq=1 ttl=56 time=26.120 ms
64 bytes from 138.68.10.168: icmp_seq=2 ttl=56 time=25.972 ms
64 bytes from 138.68.10.168: icmp_seq=3 ttl=56 time=27.232 ms

--- 138.68.10.168 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 25.972/26.955/28.494/1.014 ms
```

Now connect to the droplet. By default, you log in as as the root user:

```
$ ssh root@138.68.10.168

[...bunch of login stuff...]

root@ubuntu-s-1vcpu-2gb-sfo2-01:~#
```

Now we're ready to get started.

## Prepare for LAMP

[See DO guide](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04)

### Make Normal User

Add a non-root user:

```
adduser melo
passwd melo
usermod -aG sudo melo
```

(While we're at it, create a git user as well for gitea):

```
adduser git
passwd git
```

Prepare to SSH as that user:

```
mkdir /home/melo/.ssh
chown melo:melo /home/melo/.ssh
chmod 700 /home/melo/.ssh
chmod 600 /home/melo/.ssh/authorized_key
```

SSH as this user in a separate window (keep one window open and logged in as root!), and test sudo abilities:

```
sudo whoami
```

Disable root login via ssh:

```
vim /etc/ssh/sshd_config
```

Change `PermitRootLogin` to `no`. 

Restart SSH service:

```
sudo service ssh restart
```

Now log out and log back in as user melo.

### Dotfiles Bootstrap

[Dotfiles link](https://charlesreid1.com:3000/dotfiles/wisko/src/master/install_packages.sh)

Start out with the wisko dotfiles repository:

```
apt-get install git
cd ~
git clone https://charlesreid1.com:3000/dotfiles/wisko.git
cd wisko
```

Prepare for bootstrap:

```
./pre_bootstrap.sh
```

Now bootstrap:

```
./bootstrap.sh
```

Then set the machine name:

```
sudo ./set_machine_name.sh
```

Now log out and log back in.

### Aptitude

Install a bunch of the packages that are needed:

```
cd wisko
sudo ./install_packages.sh
```

Now we're ready to start on the LAMP server.

## Installing Apache

```
sudo apt-get install -y apache2
```

Edit Apache config file:

```
sudo vim /etc/apache2/sites-enabled/000-default.conf
```

Set the server name:

```
ServerName www.allthehatsformaps.com
...
DocumentRoot /var/www/htdocs/
```

Also make note of Apache username, should be `www-data`.

Check for syntax errors:

```
sudo apache2ctl configtest
```

Restart Apache:

```
sudo service apache2 restart
```


## Installing MySQL

```
sudo apt-get install -y mysql-server
```

This will ask you to set a root password.

Once the installation is complete, run:

```
mysql_secure_installation
```

to lock down MySQL. Do not configure `VALIDATE PASSWORD PLUGIN`.

## Installing PHP

```
sudo apt-get install -y php libapache2-mod-php php-mcrypt php-mysql php-cli
```

Make Apache look for `index.php` by editing:

```
/etc/apache2/mods-enabled/dir.conf
```

and changing:

```
DirectoryIndex index.html 
```

to this:

```
DirectoryIndex index.php index.html 
```

Restart apache:

```
sudo service apache2 restart
```

## Installing Wordpress

[boom](https://codex.wordpress.org/Installing_WordPress#Famous_5-Minute_Installation)

Download latest Wordpress:

```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz
```

This extracts to `wordpress/`. Now open MySQL and create a database for wordpress:

```
$ mysql -u root -p

mysql> CREATE DATABASE wordpress;
mysql> CREATE USER 'wpsql'@'localhost' IDENTIFIED BY "yourpasswordgoeshere";
mysql> GRANT ALL PRIVILEGES ON wordpress.* TO "wpsql"@"localhost"
```

This creates a MySQL user `wpsql` and a MySQL database called `wordpress`.

Configure Wordpress by copying example config file to actual config file:

```
cp wp-config-sample.php wp-config.php
```

Edit `wp-config.php` and change the following:
* DB_NAME
* DB_USER
* DB_PASSWORD
* DB_HOST
* DB_CHARSET
* DB_COLLATE

Use [online secret generator](https://api.wordpress.org/secret-key/1.1/salt/) to set secret key values.

Now move the wordpress folder to your web root:

```
mv wordpress /var/www/htdocs/wordpress/
```

Change permissions so wordpress directory is owned by the Apache user:

```
chown -R www-data:www-data /var/www/htdocs/wordpress
```

Visit the Wordpress site and set it up:

```
<ip-addr-of-machine>/wordpress
```

Once you have set up the Wordpress site, you should protect the `wp-config.php` file:

```
sudo chown melo:melo wp-config.php
```

### FTP Server

To upgrade Wordpress you need an ftp server running. vsftpd is a lightweight ftp server. To install it:

```
sudo apt-get install vsftpd
```

By default, the `local_enable` option is set in `/etc/vsftpd.conf`, meaning you can log in to the ftp server using system credentials.

To check and make sure the process of uploading files via FTP works, try upgrading a plugin or installing/removing a plugin.

## Installing Gitea

### Goenv: To Manage Go Version

Start by installing goenv:

```
git clone https://github.com/syndbg/goenv.git ~/.goenv
```

Now add .goenv to your path in .bash_profile so goenv is a command

```
# add these line to ~/.bash_profile
export GOENV_ROOT="$HOME/.goenv"
export PATH="$GOENV_ROOT/bin:$PATH"
```

Now source it:

```
source ~/.bash_profile
```

List latest versions of go:

```
goenv install -l
```

Pick one to install:

```
goenv install 1.7.5
goenv global 1.7.5
```

Now set this as the go version:

```
eval "$(goenv init -)"
```

You will need to execute the above command **each time you wish to use the goenv version of Go.**

(Alternatively, you can add it to `.bashrc` to run it in each new shell.)

### Gitea: Git Web Server

[Installation of gitea from source](https://docs.gitea.io/en-us/install-from-source/)

```
go get -d -u code.gitea.io/gitea
cd $GOPATH/src/code.gitea.io/gitea
```

Now check out the version of gitea that you want to use:

```
git branch -a
git checkout origin/release/v1.2
```

Build with tag `bindata`:

```
TAGS="bindata" make generate build
```

The gitea binary is entirely self-contained. Before you run the binary, create a folder for gitea to use to store repositories (this should be somewhere you have read/write access, like `~/.gitea`):

```
mkdir ~/.gitea
```

We will point gitea to this directory in the next step. To run the binary from the current directory as the current user:

```
./gitea web
```

Now you can navigate to `<server-ip-address>:3000` to set up gitea.

### Set Up Gitea (via Browser)

Visit `<server-ip-address>:3000` in the browser to set up Gitea. You will need to set up a database, and we can use MySQL again. First, create a gitea user in MySQL:

```
$ mysql -u root -p

mysql> CREATE DATABASE gitea;
mysql> CREATE USER 'giteasql'@'localhost' IDENTIFIED BY "yourpasswordgoeshere";
mysql> GRANT ALL PRIVILEGES ON gitea.* TO "giteasql"@"localhost"
```

Now you should be able to punch in all your settings. Make sure you change the address of the app from `localhost:3000` to `<server-ip-address>:3000`.

### Updating Configuration/Templates

The path where gitea is installed is here:

```
/home/melo/gocode/src/code.gitea.io/gitea
```

available as a shortcut in the home directory:

```
$ ll ~/
...
lrwxrwxrwx  1 melo melo   30 Feb 24 22:17 gitea -> gocode/src/code.gitea.io/gitea/
```

Configuration file for gitea is located here:

```
~/gitea/custom/conf/app.ini
```



If you change the config file or any page templates, you will have to re-build the binary for the changes to take effect. 

To rebuild the go binary, just set your go version with goenv and re-execute the make command from above:

```
eval "$(goenv init -)"
cd ~/gitea
TAGS="bindata" make generate build
```

### Errors

Note: if you see the following error, check which version of go you are using:

```
go build -i -v  -tags 'bindata' -ldflags '-s -w -X "main.Version=1.2.3" -X "main.Tags=bindata"' -o gitea
vendor/code.gitea.io/git/command.go:9:2: cannot find package "context" in any of:
	/home/melo/gocode/src/code.gitea.io/gitea/vendor/context (vendor tree)
	/usr/lib/go-1.6/src/context (from $GOROOT)
	/home/melo/gocode/src/context (from $GOPATH)
Makefile:205: recipe for target 'gitea' failed
make: *** [gitea] Error 1
```

When you run `which go` you should see 

```
$ which go
/home/melo/.goenv/shims/go
```

If you see this, you will have problems:

```
$ which go
/usr/bin/go
```

