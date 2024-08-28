## Setting up a LAMP stack on a Mac

Followed this tutorial and set up PHP8 instead of using the defult php7 installation:

https://deezombiedude612.github.io/wp-labs/lamp_macos/#step-3-create-usernameconf-file

Use the command `apachectl configtest` to check for errors in the httpd.conf file

Some additional work is required after of the PHP8 install. Add the LoadModule line and FilesMatch code as specified below in the terminal output.

```
==> php
To enable PHP in Apache add the following to httpd.conf and restart Apache:
    LoadModule php_module /opt/homebrew/opt/php/lib/httpd/modules/libphp.so

    <FilesMatch \.php$>
        SetHandler application/x-httpd-php
    </FilesMatch>

Finally, check DirectoryIndex includes index.php
    DirectoryIndex index.php index.html

The php.ini and php-fpm.ini file can be found in:
    /opt/homebrew/etc/php/8.3/

To start php now and restart at login:
  brew services start php
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/php/sbin/php-fpm --nodaemonize
```

Example commands to run at the terminal and output.

```
(base) ➜  apache2 brew services start php
Service `php` already started, use `brew services restart php` to restart.
(base) ➜  apache2 sudo apachectl restart   
Password:
(base) ➜  apache2 sudo apachectl configtest
[Tue Aug 27 12:54:24.351659 2024] [so:error] [pid 40204] AH06665: No code signing authority for module at /opt/homebrew/opt/php/lib/httpd/modules/libphp.so specified in LoadModule directive.
httpd: Syntax error on line 170 of /private/etc/apache2/httpd.conf: Code signing absent - not loading module at: /opt/homebrew/opt/php/lib/httpd/modules/libphp.so
```

If the httpd syntax error above is output, you need create a local Keychain Certification Authority and a code signing certificate.

https://www.simplified.guide/macos/keychain-ca-code-signing-create
https://www.simplified.guide/macos/keychain-cert-code-signing-create

After following the steps you should have 2 new items in your Keychain Access under My Certificates. A code signing certificate and a CA both with attached private keys. 

Then you need to run this line in the terminal:

```
codesign --sign "your-authority-name" --force --keychain ~/Library/Keychains/login.keychain-db /opt/homebrew/opt/php/lib/httpd/modules/libphp.so

(base) ➜  apache2 codesign --sign "Rebecca Osselton" --force --keychain ~/Library/Keychains/login.keychain-db /opt/homebrew/opt/php/lib/httpd/modules/libphp.so 
/opt/homebrew/opt/php/lib/httpd/modules/libphp.so: replacing existing signature
```

After this restart php and then Apache2.

(base) ➜  apache2 brew services restart php

Stopping `php`... (might take a while)
==> Successfully stopped `php` (label: homebrew.mxcl.php)
==> Successfully started `php` (label: homebrew.mxcl.php)

(base) ➜  apache2 sudo apachectl restart 


The locahost/phpinfo.php page should show the PHP Information page showing the version and modules etc.

If this stops working you may need to comment out the opcache path in

 `php/xxx/conf.d/ext-opcache.ini`

Restart php and apache again.

```
==> mysql
Upgrading from MySQL <8.4 to MySQL >9.0 requires running MySQL 8.4 first:
 - brew services stop mysql`
 - brew install mysql@8.4
 - brew services start mysql@8.4
 - brew services stop mysql@8.4
 - brew services start mysql

We've installed your MySQL database without a root password. To secure it run:
    mysql_secure_installation

MySQL is configured to only allow connections from localhost by default

To connect run:
    mysql -u root

To restart mysql after an upgrade:
  brew services restart mysql
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/mysql/bin/mysqld_safe --datadir\=/opt/homebrew/var/mysql
  ```