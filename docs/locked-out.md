# locked out of wisko

no problem, i got this.

```
$ ssh melo@159.65.75.41

ECDSA key fingerprint is ...

Are you sure you want to continue connecting (yes/no)? yes

Warning: Permanently added '159.65.75.41' (ECDSA) to the list of known hosts.

melo@159.65.75.41: Permission denied (publickey).

```
oh, so i have to generate a new key pair. lemme google that.

## creating a new public key

[help from do](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets)

```
ssh-keygen -t rsa
Enter file in which to save the key (/Users/melo/.ssh/id_rsa):

```
hit enter to store in user ssh root folder

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again

```
hit enter to store without passphrase

## copy key to do account

run from the command line:
```
cat ~/.ssh/id_rsa.pub

```

open settings > security in do, selection to add new key
paste in the above key and name it accordingly

## login/ add my key to the server

(**WARNING:** possibly bad to run these commands.)

```
cat ~/.ssh/id_rsa.pub | ssh melo@159.65.75.41 "cat >> /home/melo/.ssh/authorized_keys"

melo@159.65.75.41: Permission denied (publickey).
```

no? 

```
cat ~/.ssh/id\_rsa.pub | ssh root@159.65.75.41 "cat >> ~/.ssh/authorized\_keys"
root@159.65.75.41: Permission denied (publickey).

```
oh yea, charles turned off root access via command line.

## ssh debug mode w/ charles

```
ssh -vvv melo@159.65.75.41


root@159.65.75.41: Permission denied (publickey).
Melo:~ melo$ ssh -vvv melo@159.65.75.41
OpenSSH_7.6p1, LibreSSL 2.6.2
debug1: Reading configuration data /Users/melo/.ssh/config
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 48: Applying options for *
debug2: ssh_connect_direct: needpriv 0
debug1: Connecting to 159.65.75.41 port 22.
debug1: Connection established.
debug1: identity file /Users/melo/.ssh/id_rsa type 0
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/melo/.ssh/id_ed25519-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_7.6
debug1: Remote protocol version 2.0, remote software version OpenSSH_7.2p2 Ubuntu-4ubuntu2.4
debug1: match: OpenSSH_7.2p2 Ubuntu-4ubuntu2.4 pat OpenSSH* compat 0x04000000
debug3: fd 5 is O_NONBLOCK
debug1: Authenticating to 159.65.75.41:22 as 'melo'
debug3: hostkeys_foreach: reading file "/Users/melo/.ssh/known_hosts"
debug3: record_hostkey: found key type ECDSA in file /Users/melo/.ssh/known_hosts:20
debug3: load_hostkeys: loaded 1 keys from 159.65.75.41
debug3: order_hostkeyalgs: prefer hostkeyalgs: ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521
debug3: send packet: type 20
debug1: SSH2_MSG_KEXINIT sent
debug3: receive packet: type 20
debug1: SSH2_MSG_KEXINIT received
debug2: local client KEXINIT proposal
debug2: KEX algorithms: curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha256,diffie-hellman-group14-sha1,ext-info-c
debug2: host key algorithms: ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-ed25519-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ssh-ed25519,rsa-sha2-512,rsa-sha2-256,ssh-rsa
debug2: ciphers ctos: chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com
debug2: ciphers stoc: chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com
debug2: MACs ctos: umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1
debug2: MACs stoc: umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1
debug2: compression ctos: none,zlib@openssh.com,zlib
debug2: compression stoc: none,zlib@openssh.com,zlib
debug2: languages ctos: 
debug2: languages stoc: 
debug2: first_kex_follows 0 
debug2: reserved 0 
debug2: peer server KEXINIT proposal
debug2: KEX algorithms: curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1
debug2: host key algorithms: ssh-rsa,rsa-sha2-512,rsa-sha2-256,ecdsa-sha2-nistp256,ssh-ed25519
debug2: ciphers ctos: chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com
debug2: ciphers stoc: chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com
debug2: MACs ctos: umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1
debug2: MACs stoc: umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1
debug2: compression ctos: none,zlib@openssh.com
debug2: compression stoc: none,zlib@openssh.com
debug2: languages ctos: 
debug2: languages stoc: 
debug2: first_kex_follows 0 
debug2: reserved 0 
debug1: kex: algorithm: curve25519-sha256@libssh.org
debug1: kex: host key algorithm: ecdsa-sha2-nistp256
debug1: kex: server->client cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: kex: client->server cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug3: send packet: type 30
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug3: receive packet: type 31
debug1: Server host key: ecdsa-sha2-nistp256 SHA256:y1jQr+0TsjhrdM3TGWSZ/BGNdzxrK7u+hUXS/TzS1ss
debug3: hostkeys_foreach: reading file "/Users/melo/.ssh/known_hosts"
debug3: record_hostkey: found key type ECDSA in file /Users/melo/.ssh/known_hosts:20
debug3: load_hostkeys: loaded 1 keys from 159.65.75.41
debug1: Host '159.65.75.41' is known and matches the ECDSA host key.
debug1: Found key in /Users/melo/.ssh/known_hosts:20
debug3: send packet: type 21
debug2: set_newkeys: mode 1
debug1: rekey after 134217728 blocks
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug3: receive packet: type 21
debug1: SSH2_MSG_NEWKEYS received
debug2: set_newkeys: mode 0
debug1: rekey after 134217728 blocks
debug2: key: /Users/melo/.ssh/id_rsa (0x7f897241b200)
debug2: key: /Users/melo/.ssh/id_dsa (0x0)
debug2: key: /Users/melo/.ssh/id_ecdsa (0x0)
debug2: key: /Users/melo/.ssh/id_ed25519 (0x0)
debug3: send packet: type 5
debug3: receive packet: type 7
debug1: SSH2_MSG_EXT_INFO received
debug1: kex_input_ext_info: server-sig-algs=<rsa-sha2-256,rsa-sha2-512>
debug3: receive packet: type 6
debug2: service_accept: ssh-userauth
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug3: send packet: type 50
debug3: receive packet: type 51
debug1: Authentications that can continue: publickey
debug3: start over, passed a different list publickey
debug3: preferred publickey,keyboard-interactive,password
debug3: authmethod_lookup publickey
debug3: remaining preferred: keyboard-interactive,password
debug3: authmethod_is_enabled publickey
debug1: Next authentication method: publickey
debug1: Offering public key: RSA SHA256:iIOTgB1gfdbEi11gdn426XlamlH53riKdLqImE22zi8 /Users/melo/.ssh/id_rsa
debug3: send_pubkey_test
debug3: send packet: type 50
debug2: we sent a publickey packet, wait for reply
debug3: receive packet: type 51
debug1: Authentications that can continue: publickey
debug1: Trying private key: /Users/melo/.ssh/id_dsa
debug3: no such identity: /Users/melo/.ssh/id_dsa: No such file or directory
debug1: Trying private key: /Users/melo/.ssh/id_ecdsa
debug3: no such identity: /Users/melo/.ssh/id_ecdsa: No such file or directory
debug1: Trying private key: /Users/melo/.ssh/id_ed25519
debug3: no such identity: /Users/melo/.ssh/id_ed25519: No such file or directory
debug2: we did not send a packet, disable method
debug1: No more authentication methods to try.
melo@159.65.75.41: Permission denied (publickey).
```

Error message is totally unhelpful. Googling turns up some problems with file permissions and with public key access being disabled in SSH config file.

Logged in via DO, opened a console on the machine (Droplets > More > Access console), and found that the ssh folder was owned by root. Nobody can log in via ssh. Charles updated the permissions:

```
sudo chown -R melo:melo ~/.ssh
sudo chmod 700 ~/.ssh
sudo chmod 600 ~/.ssh/*
```

This still didn't work.

Finally, uninstalling/reinstalling openssh did the trick:

```
sudo apt-get purge openssh-client openssh-server
sudo apt-get install openssh-client openssh-server
```

then remove the `.ssh` folder in the root user's home directory. Regenerate the keys as the root user:

```
ssh-keygen
```

Now the machine should be ready to run the reinstalled SSH server.

To enable passwordless access, assuming you are connecting from a local machine (Mac) to a remote machine (wisko):

On the Mac: generate your ssh key:

```
ssh-keygen
```

print your public key to the screen:

```
cat ~/.ssh/id_rsa.pub
```

Copy this key to the clipboard.

On the Mac, log in to wisko:

```
ssh melo@<wisko-ip>
```

Once you are logged in, edit the following file with your favorite text editor (e.g., nano):

```
nano ~/.ssh/authorized_keys
```

Add the public key from your Mac as the last line of the file. Save it and close it.

Now test out your passwordless access: SSH from the Mac to wisko. The changes should take effect immediately.

[Link to Uncle Chucky's Passwordless SSH Login Guide](http://charlesreid1.com/wiki/SSH#Passwordless_Login)

