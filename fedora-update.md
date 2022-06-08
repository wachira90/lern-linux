# fedora update error message

````
Error: Cannot retrieve metalink for repository: fedora/20/x86_64. Please verify its path and try again
````

## change "https" to "http"
````
nano /etc/yum.repos.d/fedora.repo
nano /etc/yum.repos.d/fedora-updates.repo
````

## run command
````
# yum update ca-certificates
//or 
# yum reinstall ca-certificates
````

## change back "http" to "https"

````
nano /etc/yum.repos.d/fedora.repo
nano /etc/yum.repos.d/fedora-updates.repo
````

