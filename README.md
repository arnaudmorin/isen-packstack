#ISEN packstack

##Pré-requis

 * Installer Fedora 20

##Configuration Réseaux

Commencer par désactiver Network Manager :
```
sudo systemctl stop NetworkManager.service
sudo systemctl disable NetworkManager.service
```

Et activer le réseau standard
```
sudo systemctl enable network.service 
sudo systemctl start network.service
```

Puis le serveur SSH :
```
sudo systemctl enable sshd
sudo systemctl start sshd
```


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
NAME=eth0
ONBOOT=yes
HWADDR=52:54:00:2d:6e:61
IPADDR=192.168.42.42
NETMASK=255.255.255.0
GATEWAY=192.168.42.1
```

###ETH1

L'interface ETH1 est configurée **sans** adresse IP et intégrée dans le switch virtuel openvswitch

Contenu du fichier **/etc/sysconfig/network-scripts/ifcfg-eth1** : (attention à changer HWADDR)
```
TYPE=OVSPort
ONBOOT=yes
DEVICETYPE=ovs
DEVICE=eth1
OVS_BRIDGE=br-ex
```

##Installation PackStack

Suivre le début du [tutoriel packstack de redhat](https://openstack.redhat.com/Quickstart) en prenant le fichier **answers-allinone.txt**.

**Ne pas oublier de modifier le fichier answers-allinone.txt pour le faire correspondre aux bonnes interfaces / adresses IP**

Résumé du tuto : 

```
sudo yum update -y
sudo yum install -y https://rdo.fedorapeople.org/rdo-release.rpm
sudo yum install -y openstack-packstack
packstack --answer-file answers-allinone.txt

```

Vérifier que l'interface eth1 est bien dans le bridge br-ex d'openvswitch :
```
sudo ovs-vsctl show
```

```
    Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port "qg-74d154d0-27"
            Interface "qg-74d154d0-27"
                type: internal
        Port "eth1"
            Interface "eth1"
```

##Dashboard

###Connexion
Essayer de pointer sur [http://192.168.42.42/dashboard](http://192.168.42.42/dashboard)
Le login est **admin**. 
Le password est fourni dans le fichier **keystonerc_admin**

###Ajouter un utilisateur
Ajouter un utilisateur dans les projets Demo et Admin, avec les droits admin à chaque fois, ca évitera de se déconnecter/reconnecter à chaque fois.

###Lancer une instance
Suivre le [tutoriel packstack de redhat](https://openstack.redhat.com/Running_an_instance)

**Lancer l'instance dans le projet demo**

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

Si ping ou ssh ne fonctionne pas, **vérifier les règles de sécurités !**

###IP Flottante
Essayer d'associer une IP flotante à son instance en suivant le [tutoriel packstack de redhat](https://openstack.redhat.com/Floating_IP_range)

TODO : Il faut une passerelle sur le réseau public pour accèder à la VM.
