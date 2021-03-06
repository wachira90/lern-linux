# Install mysql on centos7
```
timedatectl set-timezone Asia/Bangkok

yum --enablerepo=centos-sclo-rh -y install rh-mysql57-mysql-server

scl enable rh-mysql57 bash

#check version
mysql -V

which mysql

nano /etc/profile.d/rh-mysql57.sh
```

## add this to file "/etc/profile.d/rh-mysql57.sh"
```
#!/bin/bash
source /opt/rh/rh-mysql57/enable
export X_SCLS="`scl enable rh-mysql57 'echo $X_SCLS'`"
```

```
nano /etc/opt/rh/rh-mysql57/my.cnf.d/rh-mysql57-mysql-server.cnf
```

## add follows within [mysqld] section

```
[mysqld]
character-set-server=utf8
sql-mode = ""
query_cache_type = 1
query_cache_limit = 256K
query_cache_min_res_unit = 2k
query_cache_size = 75M
default-time-zone = "+07:00"
```

```
systemctl start rh-mysql57-mysqld && systemctl enable rh-mysql57-mysqld
```

```
mysql_secure_installation
```
## answer is  
n, pwd, pwd, y ,n, n, y

## install phpmyadmin

```
cd /usr/share/

wget https://files.phpmyadmin.net/phpMyAdmin/4.8.3/phpMyAdmin-4.8.3-all-languages.zip

unzip phpMyAdmin-4.8.3-all-languages.zip

chown nginx:nginx -R phpMyAdmin-4.8.3-all-languages/

chmod -R 755 phpMyAdmin-4.8.3-all-languages/

ln -s /usr/share/phpMyAdmin-4.8.3-all-languages/ /usr/share/nginx/html/dbAdmin

systemctl start firewalld && systemctl enable firewalld

firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload

sudo setsebool -P httpd_can_network_connect on 
```


## check SELinux enforce

```
getenforce
```

# If it is Enforcing

```
chcon -Rt httpd_sys_content_t /path/to/www
```

## login to phpmyadmin

```
http://<hostname>/dbAdmin
```


