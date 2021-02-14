# ssh-login-pem

## GENERATE KEY
[wachira@s4-42 ~]$ ssh-keygen -t rsa -b 2048 -v

Generating public/private rsa key pair.

Enter file in which to save the key (/home/wachira/.ssh/id_rsa): 

Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in wachira.

Your public key has been saved in wachira.pub.

The key fingerprint is:

SHA256:Sg3e2pSUJ5VsF8ZMDrQxKDAbjHWk1B/M/vZNdt2RMX8 wachira@s4-42.sm.com

The key's randomart image is:

+---[RSA 2048]----+

|   +*+oo +*=+.   |

|  ...*o =o+B+  o |

|    o .++oo..   =|

|     . =o+     oE|

|      o S.      =|

|     . =  o   o +|

|      o .. . + . |

|            . .  |

|                 |

+----[SHA256]-----+

[wachira@s4-42 ~]$

## LIST KEY FILE 

ls .ssh/

id_rsa  id_rsa.pub

## COPY KEY TO TARGET

ssh-copy-id -i .ssh/id_rsa.pub wachira@192.168.4.42 -p 2211

## RENAME FILE TO *.PEM

cp .ssh/id_rsa.pub .ssh/id_rsa.pem

## TEST LOGIN TO TARGET 

sudo ssh -i .ssh/id_rsa.pub wachira@192.168.4.42 -p 2211

## PERMISSION FILE 

sudo chmod -R 400 .ssh/

sudo chmod 775 .ssh/known_hosts

sudo chown wachira:wachira .ssh/known_hosts

sudo ls -la .ssh/

## TEST LOGIN TO TARGET

ssh -i .ssh/id_rsa.pem wachira@192.168.4.42 -p 2211


ssh-copy-id -i .ssh/id_rsa.pub wachira@192.168.6.179

## CHANGE CONFIG

sudo nano /etc/ssh/sshd_config

PasswordAuthentication no

## ANOTHERWAY COPY KEY TO TARGET 

cat .ssh/id_rsa.pub | ssh wachira@192.168.6.179 "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
