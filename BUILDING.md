## Overview

These instructions will guide you in setting up a [Poudriere](https://www.freebsd.org/doc/handbook/ports-poudriere.html) repository to build and host a port tree with custom pfSense packages.

## Requirements

* An instance of FreeBSD 10.3-RELEASE x64

### Droplet setup

DigitalOcean does not currently provide any droplets running 10.3, but you can upgrade a 10.2 droplet to 10.3 with:

```sh
sudo freebsd-update -r 10.3-RELEASE upgrade
sudo freebsd-update install
sudo reboot
sudo freebsd-update install
sudo reboot
```

Also, if you're using a droplet with less than 4G of RAM, you'll likely want to add a [swap file](https://www.freebsd.org/doc/handbook/adding-swap-space.html).

## Poudriere Setup

> These instruction are based [How To Set Up a Poudriere Build System to Create Packages for your FreeBSD Servers](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-poudriere-build-system-to-create-packages-for-your-freebsd-servers)

### Install Poudriere and Nginx

```sh
sudo pkg install poudriere nginx screen git dialog4ports
```

### Create an SSL Certificate and Key

```sh
sudo mkdir -p /usr/local/etc/ssl/{keys,certs}
sudo chmod 0600 /usr/local/etc/ssl/keys
sudo openssl genrsa -out /usr/local/etc/ssl/keys/poudriere.key 4096
sudo openssl rsa -in /usr/local/etc/ssl/keys/poudriere.key -pubout -out /usr/local/etc/ssl/certs/poudriere.cert
```

### Configure Poudriere

Edit `/usr/local/etc/poudriere.conf` and make sure the following properties are set:

```sh
NO_ZFS=yes
FREEBSD_HOST=ftp://ftp.freebsd.org
POUDRIERE_DATA=${BASEFS}/data
CHECK_CHANGED_OPTIONS=verbose
CHECK_CHANGED_DEPS=yes
PKG_REPO_SIGNING_KEY=/usr/local/etc/ssl/keys/poudriere.key
URL_BASE=http://server_domain_or_IP
GIT_URL=git://github.com/j-white/FreeBSD-ports.git
```

### Configure Nginx

```sh
sudo sh -c "echo 'nginx_enable="YES"' >> /etc/rc.conf"
```

Update `/usr/local/etc/nginx/nginx.conf` with:
```
server {
    listen 80 default;
    server_name server_domain_or_IP;
    root /usr/local/share/poudriere/html;

    location /data {
        alias /usr/local/poudriere/data/logs/bulk;
        autoindex on;
    }

    location /packages {
        root /usr/local/poudriere/data;
        autoindex on;
    }
}
```

Add `log` to the `text/plain` mime type in `/usr/local/etc/nginx/mime.types`:
```
text/plain                          txt log;
```

Start nginx:
```sh
sudo service nginx start
```

### Create the jail

```sh
sudo poudriere jail -c -j freebsd_10-3x64 -v 10.3-RELEASE
```
### Create the ports tree

```sh
sudo poudriere ports -v -c -m git -B RELENG_2_3_4_MINION -p RELENG_2_3_4_MINION
```

### Configure the ports

Create the list of packages to compile:
```
sudo sh -c "echo 'net/pfSense-pkg-minion' >> /usr/local/etc/poudriere.d/pfSense-minion-packages-list"
```

Optionally:
```
sudo sh -c "echo 'lang/go' >> /usr/local/etc/poudriere.d/pfSense-minion-packages-list"
```

Set some global options:
```
sudo sh -c "echo 'OPTIONS_UNSET+= DOCS NLS X11 EXAMPLES' >> /usr/local/etc/poudriere.d/freebsd_10-3x64-make.conf"
```

Configure the port options:

```sh
sudo mkdir /usr/local/etc/poudriere.d/freebsd_10-3x64-options
sudo poudriere options -j freebsd_10-3x64 -p RELENG_2_3_4_MINION -f /usr/local/etc/poudriere.d/pfSense-minion-packages-list
```

### Build (or rebuild) the ports

```sh
sudo poudriere jail -u -j freebsd_10-3x64
sudo poudriere ports -u -p RELENG_2_3_4_MINION
screen
sudo mkdir -p /usr/ports/distfiles
sudo poudriere bulk -j freebsd_10-3x64 -p RELENG_2_3_4_MINION -f /usr/local/etc/poudriere.d/pfSense-minion-packages-list
```

## Miscellaneous Notes

### Manually building the packages

If all of the dependencies are already available on the target system, you can skip the Poudriere setup and manually build the packages using:

```sh
pkg install git
git clone git@github.com:j-white/FreeBSD-ports.git pfSense-ports
cd pfSense-ports/net/pfSense-pkg-minion/
sudo pkg install openjdk8
sudo make package
```

And install the package on the target system using:

```sh
fetch http://104.236.215.163/pfSense-pkg-minion-1.0.0.txz
pkg install pfSense-pkg-minion-1.0.0.txz
```

### Entering the jail

```sh

sudo poudriere jail -s -j freebsd_10-3x64 -p RELENG_2_3_4_MINION
jls
sudo sudo jexec $JID /bin/sh
```
