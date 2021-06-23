## RMI FOR PHP


```
yum -y remove php*
```


[4] Add Remi's RPM Repository which provides many useful packages.


```
yum  install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm -y
```


# set [priority=10]
```
sed -i -e "s/\]$/\]\npriority=10/g" /etc/yum.repos.d/remi-safe.repo
```


# for another way, change to [enabled=0] and use it only when needed
```
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/remi-safe.repo
```


# if [enabled=0], input a command to use the repository

#yum --enablerepo=remi-safe install [Package]




```
yum --enablerepo=remi-safe -y install php72 php72-php-pear php72-php-mbstring php72-php-fpm php72-php-zip php72-php-gd php72-php-fpm php72-php-mcrypt 

yum --enablerepo=remi-safe -y install php72-php-mysqli php72-php-odbc php72-php-devel 

php72-php-sqlsrv php72-php-pdo_sqlsrv
```


# load environment variables with SCL tool
```
[root@dlp ~]# scl enable php72 bash

[root@dlp ~]# php -v

PHP 7.2.1 (cli) (built: Jan 3 2018 07:51:38) ( NTS )

Copyright (c) 1997-2017 The PHP Group

Zend Engine v3.2.0, Copyright (c) 1998-2017 Zend Technologies
```


```
nano /etc/profile.d/php72.sh
```

```
#!/bin/bash

source /opt/remi/php72/enable

export X_SCLS="`scl enable php72 'echo $X_SCLS'`"
```


 
```
nano +24 /etc/opt/remi/php72/php-fpm.d/www.conf
```
# line 24: change
user = nginx

# line 26: change
group = nginx
```
systemctl restart php72-php-fpm && systemctl restart nginx
nano /usr/share/nginx/html/info.php
```
<?php
phpinfo();
?>



#if use selinux
```
chcon -Rt httpd_sys_content_t /usr/share/nginx/html
setsebool -P httpd_can_network_connect 1
setsebool -P httpd_can_network_connect_db 1
```


cd /var/opt/remi/php72/lib/ && chown nginx:nginx -R php/

https://www.tecmint.com/install-php-7-in-centos-7/

```
nano /etc/nginx/conf.d/default.conf 
# add into "server" section

location / {
     root   /usr/share/nginx/html;
     index  index.html index.htm index.php;
}

location ~ \.php$ {
    root   /usr/share/nginx/html;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    include fastcgi_params;
}
```

```
nano +904 /etc/opt/remi/php72/php.ini
```

```
date.timezone = "Asia/Bangkok"
```

```
systemctl restart nginx && systemctl restart php72-php-fpm
```

```
systemctl enable php72-php-fpm
```
