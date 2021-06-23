# SELinux

```
chcon -R -t httpd_log_t /usr/share/nginx/www.example.com/storage
chcon -R -t httpd_sys_rw_content_t /usr/share/nginx/www.example.com/temp
chcon -R -t httpd_sys_rw_content_t /usr/share/nginx/www.example.com/auth/storage/framework/sessions/
chcon -R -t httpd_sys_rw_content_t /usr/share/nginx/www.example.com/auth/storage/framework/views/
chcon -R -t httpd_log_t /usr/share/nginx/www.example.com/auth/storage/logs
```
## Installing Setools and Setroubleshoot
```
yum install setroubleshoot setools -y 
```

## SELinux Alerts
We now have a tool called sealert that analyzes the audit log used by SELinux. Sealert will scan the log file and report and will then generate a report containing all discovered SELinux issues.

To run sealert from the command-line, we need to point it to the SELinux audit log.

```
sealert -a /var/log/audit/audit.log
```


## 1. Apply the appropriate SELinux file context for Apache (httpd_sys_content_t) to the index.html file by running the following command to create a policy for it:

```
semanage fcontext -a -t httpd_sys_content_t '/webapps/app1/html/index.html'
```

## 2. With the policy created, we now can apply it to the file anytime by using the restorecon command.

```
restorecon -v '/mywebapps/app1/html/index.html'
```
