# Apache Install

https://containernerds.com

Author|Company|Email
-|-|-
Anthony Loukinas|ContainerNerds|anthony@containernerds.com



## CHAPTER 1. INTRODUCTION

We always want to start with an updated system, especially if this is freshly provisioned. Follow the process below on your respective system.

![ubuntu](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_ubuntu_386503.png) **Ubuntu**
```
$ apt-get update 
$ apt-get upgrade
$ apt-get install apache2
$ systemctl enable apache2
```

![centos](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_centos_92573.png) **CentOS**
```
$ yum update
$ yum install httpd
$ systemctl enable httpd
```

With an updated operating system and apache installed, we can now move on to some basic information and configuration.


## CHAPTER 2. INFORMATION

In this section you will find information valuable to being an apache web server administrator.

### CHAPTER 2.1. DIRECTORIES TO KNOW

These are directory locations that are critical to understand and become familar with as you begin working with web sites on your systems.

**Configuration**

System | Directory
-|-
![centos](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_centos_92573.png) CentOS|/etc/httpd
![ubuntu](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_ubuntu_386503.png) Ubuntu|/etc/apache2

**Log Directories**

System | Directory | Use
-|-|-
![centos](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_centos_92573.png) CentOS|/var/log/httpd|default log location for httpd
![ubuntu](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_ubuntu_386503.png) Ubuntu |/var/log/apache2|default log location for apache2


**Virtual Host Locations**

Directory|Description
-|-
/etc/apache2/sites-available | (Active/Disabled) Virtual Hosts
/etc/apache2/sites-enabled | (Active) Virtual Hosts

*Note: For CentOS, these directories are not created by default. We standardize this document and lesson by manually creating and including these directories in the Apache (httpd) configuration. See: Section 3.2.*

## CHAPTER 3. CONFIGURATION

### CHAPTER 3.1. SETUP ROOT WEB DIRECTORY
The default Web content root directory is `/var/www` on both CentOS and Ubuntu. It is worth noting the default virtual host that comes pre-installed points to directory `/var/www/html` for it's web content. For better description and structure we will be creating a new directory structure that makes more sense.

Create a new directory `/var/www/vhosts` by executing `sudo mkdir /var/www/vhosts`. We will use this as the default location for all websites (see: Virtual Hosts) we deploy. 

![centos](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_centos_92573.png) **CentOS**

If you are running SELinux in the `Enforcing` mode (checked by executing `sudo getenforce`), than you will need to take an extra step to give httpd full access to your new directory.

```
$ semanage fcontext -a -t httpd_sys_content_t -s system_u "/var/www/vhosts(/.*)?"
$ ls -lz /var/www
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 html
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 vhosts
```
*Note: See official [RedHat Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-selinux_contexts_labeling_files-persistent_changes_semanage_fcontext) regarding SELinux and setting directory/file permissions*

### CHAPTER 3.2. CREATE NEW VIRTUAL HOST
We will want to create a new virtual host with specific configuration to the website we will be hosting. Before we create the virtual host config, let's setup a new directory to store this new domain `domain.com`.

```bash
$ sudo mkdir -p /var/www/vhosts/domain.com/{public_html,logs,ssl}
```
*Note: The -p parameter create directories recursively if they don't exist.*

![centos](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_centos_92573.png) **CentOS**

We will need to create two directories and include them in the main `httpd.conf` file.

```bash
$ sudo mkdir /etc/httpd/{sites-available,sites-enabled}
$ vim /etc/httpd/conf/httpd.conf

# Add the at the end of the configuration file.
IncludeOptional /etc/httpd/sites-enabled/*.conf
```

Create the virtual host.

```bash
$ sudo touch /etc/httpd/sites-available/domain.com.conf
$ sudo vim /etc/httpd/sites-available.com/domain.com.conf
```
*Note: See below for sample Virtual Host*

After you've created the virtual host, we will want to enable the virtual host by soft linking it to sites-enabled:

```bash
$ sudo ln -s /etc/httpd/sites-available/domain.com.conf /etc/httpd/sites-enabled/
$ systemctl restart httpd
```

![ubuntu](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_ubuntu_386503.png) **Ubuntu**

Create the virtual host.

```bash
$ sudo touch /etc/apache2/sites-available/domain.com.conf
$ sudo vim /etc/apache2/sites-available/domain.com.conf
```
*Note: See below for sample Virtual Host*

After you've created the virtual host, we will want to enable the virtual host by soft linking it to sites-enabled:

```bash
$ sudo ln -s /etc/apache2/sites-available/domain.com.conf /etc/apache2/sites-enabled/
$ systemctl restart apache2
```
* * *

**Sample Virtual Host Configuration**
```
<VirtualHost *:80>
    ServerAdmin root@localhost
    ServerName domain.com
    ServerAlias www.domain.com

    DocumentRoot /var/www/vhosts/domain.com/public_html

    CustomLog /var/www/vhosts/domain.com/logs/access_log combined
    ErrorLog /var/www/vhosts/domain.com/logs/error_log
</VirtualHost>
```

*Note: For more configuration options see the official [Virtual Host Documentation](https://httpd.apache.org/docs/2.4/vhosts/)*

### CHAPTER 3.3. SSL CONFIGURATION

We recommend before tackling this chapter, that you've installed `certbot` and ran through our guide on generating a certificate from LetsEncrypt.

Create a new virtual host for our specific SSL configuration. This could technically be in the original virtual host hosting our insecure port 80 version of the site, but for cleanliness and understanding we will create a separate file.

![centos](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_centos_92573.png) **CentOS**

```bash
$ sudo touch /etc/httpd/sites-available/domain.com-ssl.conf
$ sudo vim /etc/httpd/sites-available/domain.com-ssl.conf
$ sudo ln -s /etc/httpd/sites-available/domain.com-ssl.conf /etc/httpd/sites-enabled/
```

![ubuntu](https://s3.amazonaws.com/bucket01.containernerds.com/icons/iconfinder_ubuntu_386503.png) **Ubuntu**

Before we create this virtual host file, let's enable the SSL mod in Apache.

```bash
$ sudo a2enmod ssl
```

```bash
$ sudo touch /etc/apache2/sites-available/domain.com-ssl.conf
$ sudo vim /etc/apache2/sites-available/domain.com-ssl.conf
$ sudo ln -s /etc/apache2/sites-available/domain.com-ssl.conf /etc/apache2/sites-enabled/
```
* * *

One thing I prefer to do, is store all SSL certificates in the website's web content directory. This is technically optional, but will affect the sample Virtual Host Configuration paths.

If using LetsEncrypt, you can soft-link the certificates.

```bash
$ sudo ln -s /etc/letsencrypt/live/domain.com/fullchain.pem /var/www/vhosts/domain.com/ssl/server.crt
$ sudo ln -s /etc/letsencrypt/live/domain.com/privkey.pem /var/www/vhosts/domain.com/ssl/server.key
```

Otherwise, place your cert & key in the ssl directory named `server.crt` and `server.key`

**Sample Virtual Host Configuration**
```
<VirtualHost *:443>
    ServerAdmin root@localhost
    ServerName domain.com
    ServerAlias www.domain.com

    DocumentRoot /var/www/vhosts/domain.com/public_html

    CustomLog /var/www/vhosts/domain.com/logs/access_log combined
    ErrorLog /var/www/vhosts/domain.com/logs/error_log

    SSLEngine on
    SSLProtocol all -SSLv2
    SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
    SSLCertificateFile "/var/www/vhosts/domain.com/ssl/server.crt"
    SSLCertificateKeyFile "/var/www/vhosts/domain.com/ssl/server.key"
</VirtualHost>
```