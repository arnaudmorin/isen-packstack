#ISEN packstack

##Pré-requis

 * Installer Fedora 20

##Configuration Réseaux

Notre platforme utilisera deux réseaux :

 * Un réseau **management** : 192.168.42.0/24
 * Un réseau **public**     : 172.0.1.0/24

Les interfaces doivent donc être bien configurées.

###ETH0

L'interface eth0 sera notre interface de management, on lui attribue une IP dans le réseau management. 
Dans mon exemple, je lui attribue l'adresse IP : **192.168.42.42**

Contenu du fichier **/etc/sysconfig/network-scripts/ifcfg-eth0** : (attention à changer HWADDR)
```
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
NAME="eth0"
ONBOOT=yes
HWADDR=52:54:00:2d:6e:61
PEERDNS=yes
PEERROUTES="yes"
IPADDR=192.168.42.42
NETMASK=255.255.255.0
BROADCAST=192.168.42.255
NETWORK=192.168.42.0
GATEWAY=192.168.42.1
USERCTL=no
PREFIX=24
```

###ETH1

L'interface ETH1 est configurée **sans** adresse IP et intégrée dans le switch virtuel openvswitch

Contenu du fichier **/etc/sysconfig/network-scripts/ifcfg-eth1** : (attention à changer HWADDR)
```
DEVICE=eth1
TYPE=OVSPort
DEVICETYPE=ovs
OVS_BRIDGE=br-ex
NM_CONTROLLED=no
ONBOOT=yes
IPV6INIT=no
USERCTL=no
```

##Installation PackStack

Suivre le début du [tutoriel packstack de redhat](https://openstack.redhat.com/Quickstart) en prenant le fichier **answer-allinone.txt**.

Résumé du tuto : 

```
sudo yum update -y
sudo yum install -y https://rdo.fedorapeople.org/rdo-release.rpm
sudo yum install -y openstack-packstack
packstack --answer-file answer-allinone.txt

```

##Dashboard

###Connexion
Essayer de pointer sur [http://192.168.42.42/dashboard](http://192.168.42.42/dashboard)
Le login est **admin**. 
Le password est fourni dans le fichier **keystonerc_admin**

###Lancer une instance
Suivre le [tutoriel packstack de redhat](https://openstack.redhat.com/Running_an_instance)

####Cas d'une instance cirros

On peut lancer une instance cirros pour aller vite. 
Pour s'y connecter, il faut entrer dans le **namespace** du router1.
Pour connaitre le namespace :

```
ip netns list
```

Ensuite, pour y entrer : 

```
sudo ip netns exec qrouter-4ab8b434-ff36-40ea-be88-a88574c5d113 bash
```

Enfin, essayer de pinger l'ip privée de la machine virtuelle cirros (10.0.0.4 dans mon cas):

```
ping 10.0.0.4
```

Puis ssh (mot de passe **cubswin:)** ) : 

```
ssh cirros@10.0.0.4
```

Si ping ou ssh ne fonctionne pas, vérifier les règles de sécurités !

###IP Flottante
Essayer d'associer une IP flotante à son instance en suivant le [tutoriel packstack de redhat](https://openstack.redhat.com/Floating_IP_range)

TODO : Il faut une passerelle sur le réseau public pour accèder à la VM.
