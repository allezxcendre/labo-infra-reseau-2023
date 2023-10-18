-Connectez-vous en SSH aux machines

je crée un utilisateur pour commencer: 

sudo useradd alexandre -m -s /bin/bash -u 2000

ssh-keygen

ssh alexandre@176.16.64.2
ssh alexandre@176.16.64.3

-🎰 Changez le nom d'hôte des machines pour avoir respectivement vm-landing1 et vm-landing2

sudo hostnamectl set-hostname vm-landing1
sudo hostnamectl set-hostname vm-landing2

-Trouvez l'adresse IP locale des machines
🎯 Quelle est l'adresse de broadcast ?

[alexandre@vm-landing1 etc]$ ip a

 Trouvez le masque de sous-réseau des machines:
 
[alexandre@vm-landing1 /]$ sudo nano etc/sysconfig/network-scripts/ifcfg-enp0s8

ETMASK=255.255.255.0

🎰 Trouvez l'adresse MAC des machines
[alexandre@vm-landing1 /]$ ip a

08:00:27:b2:b7:a2
08:00:27:a2:a4:f8

🎰 Pingez l'adresse publique du site www.ynov.com avec une des deux machines

[alexandre@vm-landing1 /]$ ping ynov.com
PING ynov.com (104.26.10.233) 56(84) bytes of data.
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=1 ttl=56 time=20.3 ms

Sur chaque machine, créez respectivement un utilisateur labo-user1 et labo-user2.

[alexandre@vm-landing1 /]$ sudo useradd labo-user1
[alexandre@vm-landing2 ~]$ sudo useradd labo-user2

🎰 Essayez de lire le contenu du fichier /var/log/tallylog
🎯 Que se passe-t-il ? Pourquoi ?

Cela ne m'ouvre pas de fichier car je ne suis pas autorisé

Ajoutez ces utilisateurs au groupe wheel et retentez
🎯 Que se pase-t-il ? Pourquoi ?
[alexandre@vm-landing2 ~]$ sudo usermod -aG wheel labo-user1
[alexandre@vm-landing2 ~]$ sudo usermod -aG wheel labo-user2

je ne peux toujours pas l'ouvrir parsque mon user n'est pas dans les admins

Retentez avec la commande sudo cat
🎯 Il n'y aura plus d'erreur. Pourquoi ?

il y a une erreur car je suis encore pas autorisé

Retentez en changeant d'utilisateur et en passant root
🎯 Ca devrait marcher. Pourquoi ?

car avec mon user root j'ai accés au commande d'admin

Ajoutez l'utilisateur labo-user1 au sudoers (petit coup de Google), puis retentez la commande sudo cat
🎯 Ca devrait marcher. Pourquoi ?

[labo-user1@vm-landing1 /]$ sudo cat var/log/tallylog
car j'ai donner accés au commande admin desormais


🎰 Pingez vm-landing2 avec vm-landing1

[labo-user1@vm-landing1 /]$ ping 172.16.64.3
PING 172.16.64.3 (172.16.64.3) 56(84) bytes of data.
64 bytes from 172.16.64.3: icmp_seq=1 ttl=64 time=2.79 ms
64 bytes from 172.16.64.3: icmp_seq=2 ttl=64 time=0.899 ms
64 bytes from 172.16.64.3: icmp_seq=3 ttl=64 time=0.517 ms
^X64 bytes from 172.16.64.3: icmp_seq=4 ttl=64 time=1.04 ms

Installez les paquets sl, dnsmasq et htop. 🎰 Vérifiez leur version

[labo-user1@vm-landing1 /]$ sudo dnf install sl
[labo-user1@vm-landing1 /]$ sudo dnf install dnsmasq
[labo-user1@vm-landing1 /]$ sudo dnf install htop
sl --version
dnsmassq --version
htop --version


Changez le DNS pour celui de CloudFlare puis 🎰 pingez google.com

[labo-user1@vm-landing1 sysconfig]$ sudo nano network-scripts/ifcfg-enp0s8
DNS=1.1.1.1
[labo-user1@vm-landing1 sysconfig]$ ping google.com
PING google.com (142.250.179.110) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=115 time=31.1 ms

Installez les paquets htop, fail2ban et unzip. 🎰 Vérifiez leur version

[alexandre@vm-landing2 ~]$ sudo dnf install htop fail2ban unzip
htop --version
fail2ban --version
unzip --version

Changez le DNS pour celui de Google puis 🎰 pingez cloudflare.com

[alexandre@vm-landing2 network-scripts]$ sudo nano icfg-enp0s8

DNS=8.8.8.8

[alexandre@vm-landing2 network-scripts]$ ping cloudflare.com
PING cloudflare.com (104.16.133.229) 56(84) bytes of data.
64 bytes from 104.16.133.229 (104.16.133.229): icmp_seq=1 ttl=56 time=13.7 ms

🎰 Trouvez et affichez la route par défaut présente sur la machine

[alexandre@vm-landing2 network-scripts]$ ip route
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
default via 172.16.64.1 dev enp0s8 proto static metric 101
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
172.16.64.0/24 dev enp0s8 proto kernel scope link src 172.16.64.3 metric 101


Sur la machine landing-vm1, changez la carte réseau en Host-Only.
🎯 Quelle est l'utilité de ce type de carte réseau ?

La VM se verra attribuer une adresse IP car il aura une connexion privé entre la carte host et la machine 

 Pingez landing-vm2 avec landing-vm1, que se passe-t-il ?
 
[labo-user1@vm-landing1 sysconfig]$ ping 172.16.64.3
PING 172.16.64.3 (172.16.64.3) 56(84) bytes of data.
64 bytes from 172.16.64.3: icmp_seq=1 ttl=64 time=0.820 ms
64 bytes from 172.16.64.3: icmp_seq=2 ttl=64 time=0.731 ms

la connexion est plus rapide