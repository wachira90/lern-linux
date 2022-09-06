# basic ubuntu

## How to Change the Date & Time

````
timedatectl set-time 21:45:53
timedatectl set-time 2019-04-10
````

## set NTP
````
timedatectl set-ntp yes
timedatectl set-ntp no
````


## Set a Timezone in Ubuntu
````
timedatectl list-timezones
timedatectl set-timezone Asia/Bangkok
````

## Set Universal Time (UTC) in Ubuntu
````
timedatectl set-timezone UTC
````

## Sync Hardware Clock

### Set Hardware Clock to Sync to Local Timezone

````
timedatectl set-local-rtc 1
````

### Set the Hardware Clock to Sync with UTC
````
timedatectl set-local-rtc 0
````

## change time

````
sudo date -s YY-MM-DD HH:MM:SS
````
