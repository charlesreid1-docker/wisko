# wisko gitea

## before you begin: gitea setup info

(see [wisko setup](https://hackmd.io/s/BkfqUw0wz) for details on how gitea was set up).

### where is gitea

The wisko gitea instance is installed to `$GOPATH/src/code.gitea.io/gitea` and there is a symlink to this directory at `~/gitea`.

This directory is actually a cloned git repo, so you can see what files you've modified from the default by running a `git status`.

The binary is in that folder, at `bin/gitea`.

### where is go

Go is installed using `goenv`, so the global version of go that goenv will set can be printed using:

```
goenv global
```

You can verify you have the goenv go installed by running:

```
which go
```

which should return `~/.goenv/shims/go`, and 

```
go version
```

which should return 1.7.5.

## task summary

Making gitea available via the subdomain `git.smallmelo.com` requires setting up a couple of components:
* subdomain
* web server to handle redirects
* gitea configuration to tell gitea where it is hosted
* https certificates valid for the subdomain

## set up git subdomain

First thing is, we need to set up the subdomain `git.smallmelo.com` to redirect to the server. This depends on the hosting provider, but should be an A Name Record:

**A Record**:

-   Record: git
-   Value: <your ip address>

## apache web server settings

Setting up a name record for the subdomain will result in requests for "git.smallmelo.com" and "smallmelo.com" to both go to your server. The server will be able to see what domain/subdomain was requested in each request, so we need to set up rules to handle the subdomain correctly.

If the git subdomain is not set up with the DNS provider yet, you can still test it locally. From whatever machine you're connecting to wisk as, add an entry to `/etc/hosts` that will map the subdomain to wisko's IP:

```
vim /etc/hosts
```

and add

```
159.65.75.41 git.smallmelo.com
```

(Do this on your machine, not on wisko.)

### apache configuration: http

Now we'll set up the Apache web server to redirect any http requests to `git.smallmelo.com` on port 80 on to gitea at port 3000.

Create a new apache config for this site at `/etc/apache2/sites-available/git-smallmelo.conf` that will contain all our settings. Set up a listener on port 80, all interfaces, that looks for requests to `git.smallmelo.com`:

**`/etc/apache2/sites-available/git-smallmelo.conf`**:

```
<VirtualHost *:80>
	ServerName git.smallmelo.com
	ServerAdmin melo@smallmelo.com

	# -------------------------
	# Need to enable the following mods:
	# a2enmod proxy
	# a2enmod proxy_http

	# Preserve original request (git.smallmelo.com)
	ProxyPreserveHost On

	# Set up proxy
	<Proxy *>
		Order allow,deny
		Allow from all
	</Proxy>
	ProxyPass / http://localhost:3000/
	ProxyPassReverse / http://localhost:3000/

	# -------------------------

	ErrorLog ${APACHE_LOG_DIR}/git-smallmelo-error.log
	CustomLog ${APACHE_LOG_DIR}/git-smallmelo-access.log combined

	# Note that we don't set a document root
	# or permissions for a web directory
	# because we're just forwarding traffic
	# to gitea, which runs an entirely separate
	# web server.

</VirtualHost>
```

### apache configuration: https

We also need to set up Apache to redirect https requests to `git.smallmelo.com` on port 443 to gitea at port 3000.

This will require an HTTPS certificate for `git.smallmelo.com`, so fire up lets encrypt.

### making apache certificate

Create an https apache config for this site at `/etc/apache2/sites-available/git-smallmelo-le-ssl.conf` that will contain all our settings. Set up a listener on port 443, all interfaces, that looks for requests to `git.smallmelo.com`.

Run the certbot:

```
$ sudo certbot certonly --non-interactive --agree-tos --email "melo@smallmelo.com" --apache -d "git.smallmelo.com"
```

This will take a minute, but should conclude with:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/git.smallmelo.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/git.smallmelo.com/privkey.pem
   Your cert will expire on 2018-06-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

### apache configuration: https (revisited)

Note that once the HTTPS request hits the server, the proxy does not need to happen over HTTPS since it happens within a single machine.

**`/etc/apache2/sites-available/git-smallmelo-le-ssl.conf`**:

```
<IfModule mod_ssl.c>
	<VirtualHost *:443>
		ServerName git.smallmelo.com
		ServerAdmin melo@smallmelo.com

		ErrorLog ${APACHE_LOG_DIR}/git-smallmelo-ssl-error.log
		CustomLog ${APACHE_LOG_DIR}/git-smallmelo-ssl-access.log combined

		# -------------------------
		# Need to enable the following mods:

		# a2enmod proxy
		# a2enmod proxy_http

		SSLEngine on
		SSLProxyEngine On
		Include /etc/letsencrypt/options-ssl-apache.conf
		SSLCertificateFile /etc/letsencrypt/live/git.smallmelo.com/fullchain.pem
		SSLCertificateKeyFile /etc/letsencrypt/live/git.smallmelo.com/privkey.pem
		
		# Preserve original request (git.smallmelo.com)
		ProxyPreserveHost On

		# Set up proxy
		<Proxy *>
			Order allow,deny
			Allow from all
		</Proxy>
		ProxyPass / http://localhost:3000/
		ProxyPassReverse / http://localhost:3000/

		# -------------------------

	</VirtualHost>
</IfModule>
```

### enable and activate

Now enable the mods:
```
a2enmod proxy
a2enmod proxy_http
```

Now enable the site:

```
a2ensite git-smallmelo
a2ensite git-smallmelo-le-ssl
```

Now activate the site:

```
service apache2 reload
```

Once the A Name DNS record propagates, you can test out the Apache configuration.

Run a dummy HTTP web server on port 3000:

```
cat "<h2>hello world</h2>" > index.html
python3 -m http.server 3000
```

Now if you visit the subdomain:

```
http://git.smallmelo.com
```

you should see a hit in the window running the HTTP server on port 3000.


## gitea configuration

### gitea version

Start out by checking which version of gitea you have. We have checked out release `1.2`, which means that our git repo is in a "headless" state. Any changes that we make are going to go into a dead end.

But that's perfect - we can keep track of all our changes relative to version 1.2, and revert any changes we don't want anymore.

Make the following changes to the default `app.ini` gitea config file:

```
RUN_USER = melo

DOMAIN = git.smallmelo.com
ROOT_URL = %(PROTOCOL)s://%(DOMAIN)s/
```

(Note, you can use git diff to pick up the changes in the file.)

Kill any instances of gitea that are already running on port 3000, and start up the gitea server:

```
cd ~/gitea
./gitea web
```

(or, do it in a screen, [as mentioned in the wisko setup doc](https://hackmd.io/s/BkfqUw0wz#Installing-Gitea)).

## https certs

See the above modifications to the document about certificates. A new SSL certificate for the git subdomain (`git.smallmelo.com`) was needed, so we used certbot to make it.

To explain the HTTPS certificate setup for git.smallmelo.com a bit more:

* Gitea runs its own web server, hosts its own files, and does not require an Apache server to run. We're just using Apache for convenience.
* Just as HTTP requests to smallmelo.com always go to port 80, where there is an Apache server listening, HTTPS requests to smallmelo.com always go to port 443, where there is also an Apache server listening
* HTTPS requests are handled by the Apache web server, so the certificates to create an encrypted session are handled by the Apache web server
* Apache must have an SSL certifiate for `git.smallmelo.com` to make the connection, or the browser will complain that the server is not configured correctly
* When we make a request for a gitea resource (say, `https://git.smallmelo.com/melo/my-cool-project`), the subdomain tells Apache where to forward the request (to gitea on port 3000), and the rest of the url (`/melo/my-cool-project`) is passed on to gitea, unmodified
* The request from Apache to gitea happens on a single machine, so the connection does not need to be encrypted, hence Apache's proxy requests go to `http://localhost:3000` and not `https://localhost:3000`

