# netplan network

## open file

sudo nano /etc/netplan/01-netcfg.yaml

## dhcp original ok

/etc/netplan/01-netcfg.yaml
````
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: yes
      dhcp6: yes
````

## static

/etc/netplan/01-netcfg.yaml
````
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: no
      addresses:
        - 192.168.121.199/24
      gateway4: 192.168.121.1
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
````

## restart service

````
sudo netplan apply
````
