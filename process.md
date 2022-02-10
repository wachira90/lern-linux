# process

## find by name
````
ps aux | grep -i firefox
````
## save process
````
top -b -n1 > /tmp/process.log
````
## find by name
````
pidof firefox
````
## find by username and services
````
pgrep -u root sshd
````

## by name using pgrep
````
pgrep -u root sshd
````

## Just look up pid for firefox process
````
pgrep firefox
````

## kill service 

````
kill -9 <pid number>
````

## mail notify

````
top -b -n1 | mail -s 'Process snapshot' you@example.com
````

