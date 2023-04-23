# adduser group

## adduser

````
adduser wachira
````

## change or set password 

````
passwd wachira
````

## adduser to groups

````
usermod -a -G wheel wachira
````

## remove groups from user

````
gpasswd -d wachira wheel
````
