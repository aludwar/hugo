---
title: Deploying ownCloud 9 on Fedora 23
author: Andrew Ludwar
type: post
date: 2016-03-16T23:14:21+00:00
url: /blog/2016/03/16/deploying-owncloud-9-on-fedora-23/
xyz_smap:
  - 1
dsq_thread_id:
  - 5075799977
categories:
  - cloud
  - open source
tags:
  - configuration management
  - open cloud
  - open source
  - public cloud

---
I&#8217;ve been playing around with server/VM provisioning lately and have come across a need to store some basic deployment files in the cloud. For me, these are things like ansible code, a home directory .tar file, among a few other personal items. With ownCloud 9 being recently released, I thought I should give it a try. Digging further into its features I see that ownCloud can federate many external cloud storage solutions (Dropbox, GoogleDrive, etc.) &#8211; cool! Another useful feature is that ownCloud can enable encryption on any personal data in addition to the encryption methods used by Dropbox and Google. This will provide an additional layer of encryption in case that Dropbox or Google&#8217;s data is compromised and decrypted, my data will still be protected by my own encryption keys. That adds a nice layer of safety for my personal data, so lets do a deployment and connect ownCloud to GoogleDrive to start.

The first thing I started with was a fresh install of Fedora 23 server. (I originally tried CentOS but ran into some out-of-date package issues with nss). With ownCloud 9 being so new, it&#8217;s not yet included in Fedora&#8217;s standard repos, so I did an install from source. You can download the source from [owncloud.org][1], verify with MD5/SHA256 and/or GPG, and continue with my steps below. I used the following two links to help with my install. [Manual installation of ownCloud][2], and [ownCloud installation from the command line][3]. Firstly, ownCloud uses a web and database server on the back end. You&#8217;re free to chose your own flavours of each, for me I chose apache and mysql(mariadb). When installing from source, you&#8217;ll need to manually install your web and database server prior to ownCloud:

<pre>dnf install httpd mysql mysql-server</pre>

Also, there&#8217;s a whack of PHP modules you&#8217;ll need. I cheekily grabbed a list of these from the Fedora packaged ownCloud 8 installation &#8211; it parses all the dependencies for you. I issued a dnf install owncloud command to get all the PHP dependencies, then awk&#8217;d them out of the output. I put them in a file, and installed with a for loop:

<pre>[root@owncloud ~]# cat /root/phprpms:
php
php-Assetic
php-PHPMailer
php-PsrLog
php-aws-sdk
php-bantu-ini-get-wrapper
php-channel-dropbox-php
php-channel-phpseclib
php-channel-sabredav
php-cli
php-common
php-doctrine-annotations
php-doctrine-cache
php-doctrine-collections
php-doctrine-common
php-doctrine-dbal
php-doctrine-inflector
php-doctrine-lexer
php-dropbox-php-Dropbox
php-gd
php-getid3
php-google-apiclient
php-guzzle-Guzzle
php-intl
php-ircmaxell-random-lib
php-ircmaxell-security-lib
php-ldap
php-lessphp
php-mbstring
php-mysqlnd
php-natxet-cssmin
php-opencloud
php-pdo
php-pear
php-pear-Auth-SASL
php-pear-Cache-Lite
php-pear-DB
php-pear-HTTP-OAuth
php-pear-HTTP-Request2
php-pear-Log
php-pear-MDB2
php-pear-Mail
php-pear-Net-SMTP
php-pear-Net-Socket
php-pear-Net-URL2
php-pecl-jsonc
php-pecl-oauth
php-pecl-zip
php-phpseclib-crypt-aes
php-phpseclib-crypt-base
php-phpseclib-crypt-blowfish
php-phpseclib-crypt-des
php-phpseclib-crypt-hash
php-phpseclib-crypt-random
php-phpseclib-crypt-rc4
php-phpseclib-crypt-rijndael
php-phpseclib-crypt-tripledes
php-phpseclib-crypt-twofish
php-phpseclib-math-biginteger
php-phpseclib-net-sftp
php-phpseclib-net-ssh2
php-pimple-lib
php-process
php-sabre-dav
php-sabre-vobject
php-sabredav-Sabre
php-sabredav-Sabre_VObject
php-scssphp
php-symfony-browser-kit
php-symfony-class-loader
php-symfony-common
php-symfony-config
php-symfony-console
php-symfony-css-selector
php-symfony-debug
php-symfony-dependency-injection
php-symfony-dom-crawler
php-symfony-event-dispatcher
php-symfony-expression-language
php-symfony-filesystem
php-symfony-finder
php-symfony-http-foundation
php-symfony-http-kernel
php-symfony-process
php-symfony-routing
php-symfony-var-dumper
php-symfony-yaml
php-twig
php-xml
t1lib
[root@owncloud ~]# for i in `cat /home/aludwar/phprpms`; do dnf install $i -y; done</pre>

That should take care of the prerequisites (you can compared with ownCloud&#8217;s docs, and php -m | grep &#8216;module&#8217; if you like). Then I enabled and started each service:

<pre>systemctl enable httpd
systemctl enable mariadb
systemctl start httpd
systemctl start mariadb</pre>

For additional security, I&#8217;ll install SSL capability on the web server. First by installing mod_ssl, then generating a self-signed SSL certificate, and adding it to the apache config:

<pre>dnf install mod_ssl
cd /etc/pki/tls/certs/
openssl req -newkey rsa:4096 -nodes -keyout owncloud.ludwar.ca.key -x509 -days 3650 -out owncloud.ludwar.ca.crt
vi /etc/httpd/conf.d/ssl.conf
...
SSLCertificateFile /etc/pki/tls/certs/owncloud.ludwar.ca.crt
...
SSLCertificateKeyFile /etc/pki/tls/certs/owncloud.ludwar.ca.key

systemctl restart httpd</pre>

Also, it&#8217;s a good idea to run the secure installation/setup of mysql(mariadb), which you can do by running this utility and following the prompts:

<pre>mysql_secure_installation</pre>

Ok! Now after a quick check of everything working, we&#8217;re ready to start installing ownCloud. To start we&#8217;ll untar the files into the default web directory, and add the owncloud config into apache:

<pre>cp -rp owncloud-9.0.0.tar /var/www/html/
tar -xvf owncloud-9.0.0.tar
vi /etc/httpd/conf.d/owncloud.conf              (add in the below)
Alias /owncloud "/var/www/html/owncloud/"

&lt;Directory /var/www/html/owncloud/&gt;
 Options +FollowSymlinks
 AllowOverride All

&lt;IfModule mod_dav.c&gt;
 Dav off
 &lt;/IfModule&gt;

SetEnv HOME /var/www/html/owncloud
SetEnv HTTP_HOME /var/www/html/owncloud

&lt;/Directory&gt;</pre>

Then we&#8217;ll need to make some SELinux permission changes. I highly recommend you DO NOT disable SELinux. Depending on your setup, there may be [additional SELinux changes][4] you&#8217;ll need to make.

<pre>semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/owncloud/data'
restorecon '/var/www/html/owncloud/data'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/owncloud/config'
restorecon '/var/www/html/owncloud/config'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/owncloud/apps'
restorecon '/var/www/html/owncloud/apps'</pre>

My data directory is on an NFS share from another computer, so I need to make an additional SELinux change:

<pre>setsebool -P httpd_use_nfs on</pre>

Now, onto the database initialization. The below install command is right out of ownCloud&#8217;s docs, make sure your default web directory is owned by the httpd daemon user, and run the install command:

<pre>cd /var/www/html/owncloud/
chown -R apache:apache /var/www/html/owncloud/
sudo -u apache php occ  maintenance:install --database "mysql" --database-name "owncloud" --database-user "root" --database-pass "password" --admin-user "admin" --admin-pass "password"
ownCloud is not installed - only a limited number of commands are available
ownCloud was successfully installed</pre>

Now you should be able to browse to _https://localhost/owncloud_ to login! Depending on if you&#8217;re accessing the owncloud host locally or from another server, you may need to add the domain to the trusted domains list in _/var/www/html/owncloud/config/config.php_. The web GUI should alert and prompt you to do that, but if not, you can modify the file directly. Here&#8217;s my config.php file with the password data omitted:

<pre>[root@owncloud conf.d]# cat /var/www/html/owncloud/config/config.php
&lt;?php
$CONFIG = array (
 ...
 'trusted_domains' =&gt; 
 array (
 0 =&gt; 'localhost',
 1 =&gt; '192.168.1.100',
 2 =&gt; 'owncloud.ludwar.ca',
 ),
 'datadirectory' =&gt; '/var/www/html/owncloud/data',
 'overwrite.cli.url' =&gt; 'http://localhost',
 'dbtype' =&gt; 'mysql',
 'version' =&gt; '9.0.0.19',
 'dbname' =&gt; 'owncloud',
 'dbhost' =&gt; 'localhost',
 'dbtableprefix' =&gt; 'oc_',
 ...);
[root@owncloud conf.d]#</pre>

Once in the GUI, you can enable encryption from the admin menu at the far right. Server side encryption can be enabled with a checkbox. It&#8217;s also good to enter a recovery key in the event you need to recover your encrypted data:<figure id="attachment_313" aria-describedby="caption-attachment-313" style="width: 800px" class="wp-caption alignnone">

[<img class="size-full wp-image-313" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt1.png" alt="ownCloud encryption module" width="800" height="289" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt1.png 800w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt1-300x108.png 300w" sizes="(max-width: 800px) 100vw, 800px" />][5]<figcaption id="caption-attachment-313" class="wp-caption-text">ownCloud encryption configuration</figcaption></figure> 

Then you need to install  the default encryption app. First at the top left, switch from files to apps, then click on not enabled, and find and enable the default encryption module. You may need to logout and back in again for the keys to initialize.<figure id="attachment_311" aria-describedby="caption-attachment-311" style="width: 800px" class="wp-caption alignnone">

[<img class="size-full wp-image-311" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt.png" alt="ownCloud encrypt application" width="800" height="429" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt.png 800w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt-300x161.png 300w" sizes="(max-width: 800px) 100vw, 800px" />][6]<figcaption id="caption-attachment-311" class="wp-caption-text">ownCloud encrypt application</figcaption></figure> 

Now, to add external storage from Google Drive, you&#8217;ll need to enable API access and create OAuth2 credentials in your Google account. You can do this by visiting https://console.developers.google.com/apis/credentials, and creating a project, then creating OAuth2 credentials within that project. This will give you a ClientID and Client Secret to enter into ownCloud. ownCloud will access your Google Drive data via Google&#8217;s API. Here&#8217;s what mine looks like, and I also needed to add my domain and a redirect URI for Google to redirect the user (web browser) after authenticating:

[<img class=" wp-image-315" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-1024x226.png" alt="Google Dev Console" width="802" height="177" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-1024x226.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-300x66.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console.png 1200w" sizes="(max-width: 802px) 100vw, 802px" />][7]

Google Dev Console<figure id="attachment_316" aria-describedby="caption-attachment-316" style="width: 800px" class="wp-caption alignnone">

[<img class="size-full wp-image-316" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-2.png" alt="Google Dev Console" width="800" height="529" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-2.png 800w, https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-2-300x198.png 300w" sizes="(max-width: 800px) 100vw, 800px" />][8]<figcaption id="caption-attachment-316" class="wp-caption-text">Google Dev Console</figcaption></figure> 

In the admin section again, you can enter those Client ID and secret details, add the user you&#8217;re giving access to the storage, and click grant access:<figure id="attachment_322" aria-describedby="caption-attachment-322" style="width: 806px" class="wp-caption alignnone">

[<img class=" wp-image-322" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage-1024x524.png" alt="ownCloud external storage" width="806" height="412" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage-1024x524.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage-300x154.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage.png 1200w" sizes="(max-width: 806px) 100vw, 806px" />][9]<figcaption id="caption-attachment-322" class="wp-caption-text">ownCloud external storage</figcaption></figure> 

Now login as that user, and your Google Drive files will be available!<figure id="attachment_323" aria-describedby="caption-attachment-323" style="width: 720px" class="wp-caption alignnone">

[<img class="size-large wp-image-323" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage2-1024x274.png" alt="ownCloud Google Drive" width="720" height="193" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage2-1024x274.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage2-300x80.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage2.png 1200w" sizes="(max-width: 720px) 100vw, 720px" />][10]<figcaption id="caption-attachment-323" class="wp-caption-text">ownCloud Google Drive</figcaption></figure> <figure id="attachment_324" aria-describedby="caption-attachment-324" style="width: 720px" class="wp-caption alignnone">[<img class="size-large wp-image-324" src="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage3-1024x287.png" alt="ownCloud Google Drive" width="720" height="202" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage3-1024x287.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage3-300x84.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage3.png 1200w" sizes="(max-width: 720px) 100vw, 720px" />][11]<figcaption id="caption-attachment-324" class="wp-caption-text">ownCloud Google Drive</figcaption></figure> 

This ended up being relatively long, so I&#8217;ll cover further work in an additional post. 🙂

 [1]: https://owncloud.org/install/
 [2]: https://doc.owncloud.org/server/9.0/admin_manual/installation/source_installation.html
 [3]: https://doc.owncloud.org/server/9.0/admin_manual/installation/command_line_installation.html
 [4]: https://doc.owncloud.org/server/9.0/admin_manual/installation/selinux_configuration.html
 [5]: https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt1.png
 [6]: https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-encrypt.png
 [7]: https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console.png
 [8]: https://calgaryrhce.ca/wp-content/uploads/2016/03/google-dev-console-2.png
 [9]: https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage.png
 [10]: https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage2.png
 [11]: https://calgaryrhce.ca/wp-content/uploads/2016/03/owncloud-extstorage3.png