# firewall ufw command

## check service 

````
ufw status
````

## start service 

````
ufw enable
````

## allow port 22

````
ufw allow 22
ufw allow 22/tcp
ufw allow ssh
ufw allow ssh/tcp
````

## deny port 22

````
ufw deny 22
ufw deny 22/tcp
ufw deny ssh
ufw deny ssh/tcp
````

## example

````
ufw deny from 192.168.2.100/8 to 192.168.2.101 port 25
````

## Let’s look at the limit option. If you have any reason for concern that someone might be attempting a denial of service attack on your machine, via port 80. You can limit connections to that port with UFW, like so:

````
sudo ufw limit 80/tcp
````

## example

````
sudo ufw allow out on eth0 to any port 25 proto tcp
sudo ufw deny in on eth0 from any 25 proto tcp
````


## available arguments, the ones you’ll use the most with the ufw command are:

````
allow

deny

reject

limit

status: displays if the firewall is active or inactive

show: displays the current running rules on your firewall

reset: disables and resets the firewall to default

reload: reloads the current running firewall

disable: disables the firewall
````
