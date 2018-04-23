# Installing Geoserver & Connecting Over HTTPS

[Installing Geoserver instructions](http://docs.geoserver.org/stable/en/user/installation/linux.html)

## Install JRE 8

[install jre 8](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)

```
sudo apt-get update
sudo apt-get install default-jre
```
## Create geoserver folder 

in `/usr/share`:
```
mkdir geoserver
```

## Downloaded geoserver on my computer, then ftp it to wisko

* Download geoserver files on my computer
* unzip
* copy files from the geoserver folder onto wisko:

from my computer:
```
scp -r /Users/melo/Desktop/geoserver-2.13.0  melo@159.65.75.41:/usr/share/geoserver
```

permission denied. fixed with:

```
sudo chown -R $USER:$USER /usr/share/geoserver

```

but then i decided to use FileZilla to transfer the files once I fixed my pemissions. Maybe another day, scp.


## add env variable
`echo "export GEOSERVER_HOME=/usr/share/geoserver" >\> ~/.profile`

## make myself the owner

`sudo chown -R USER_NAME /usr/share/geoserver/`

## start geoserver

```
cd /usr/share/geoserver/bin
sh startup.sh

```

--------

## setting up certs

To enable SSL connections to the geoserver, let Apache handle the SSL connections, and then pass the requests on to geoserver locally.

Apache already has certificates living at:

```
/etc/letsencrypt/live/smallmelo.com
```

and an HTTPS configuration file at:

```
/etc/apache2/sites-avaiable/smallmelo-le-ssl.conf
```

Modify the SSL configuration file to accept any URLs ending in `geoserver/` and forward them to the geoserver on port 8080.

There should be a `<VirtualHost>` block that controls the settings for HTTPS requests. Add a `<Location>` block to configure the behavior for geoserver URLs (note the order):

```
<VirtualHost *:443>
    
    ...
    
    # This block is new:
    <Location /geoserver>
        ProxyPass         http://localhost:8080/geoserver
        ProxyPassReverse  http://localhost:8080/geoserver
        Order allow,deny
        Allow from all
    </Location>
    
    ...
    
    # This must come after /geoserver block
    <Directory /var/www/smallmelo/public_html>
        ...
    </Directory>
    
</Location>
```

Note that the reverse proxy we have set up will forward requests *unencrypted* to the geoserver at port 8080, but because this happens within a single machine, there is no need to encrypt it.

Now ensure the proxy mod is installed:

```
$ a2enmod proxy
```

and restart the server:

```
$ sudo service apache2 restart
```

One other thing we had to do was ensure that Jetty/geoserver were only listening for requests locally. This requies binding Jetty to the IP address 127.0.0.1.

To do this, we modify one parameter in `app.ini`:

```
jetty.host=127.0.0.1
```

Restart geoserver:

```
killall geoserver && sh /usr/share/geoserver/bin/startup.sh
```

## Finishing Touches

Because Geosever was setup in order to be used in a dev environment for an xcode project, we needed to be able to ping a wms service from the site.  

The attempt resulted in [an error](https://gist.github.com/Smallmelo/acb41ea71dea6ca64b0652973ed04158).

[This line](https://gist.github.com/Smallmelo/acb41ea71dea6ca64b0652973ed04158#file-gistfile1-txt-L2572) demonstrates that your network trace is seeing "localhost:8080", which it should not. So, it's an indication that the server might be returning URLs to resources that say "localhost:8080"

### The solution:
Update the proxy url in Geoserver's Global Settings:
![Geoserver proxy settings](https://i.imgur.com/pQYIT0v.png)


