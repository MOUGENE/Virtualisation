# TP1
# Configuration d'un Réseau pour les VMs

## Connexion à la VM Ubuntu
```bash
ssh -p 2222 nom_vm_ubuntu@127.0.0.1
```

## Création d'un Nouveau Réseau pour les VMs
Pour créer un nouveau réseau pour les VMs, ping l'adresse de la VM renseignée sur l'adresse IPv4 de l'hôte. 

Exemple pour l'addr2 :
```plaintext
192.168.0.1
```

En tapant dans le cmd `ipconfig`, l'addr2 créée est la carte ethernet3 dans mon cas.

## Configuration du Pare-feu Windows
Pour pouvoir ping depuis la VM, il faut activer certaines règles dans le pare-feu Windows :

1. Panneau de configuration
2. Pare-feu
3. Paramètres avancés
4. Analyse de l'ordinateur virtuel (Demande d'écho - Trafic entrant ICMPv4)
5. Activer la règle

## Ajout d'une 2e Carte Réseau sur la VM
Pour ajouter la 2e carte réseau sur la VM, ajouter l'interface réseau sur VirtualBox et ensuite faire les commandes et les modifications suivantes sur la VM :

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Ajouter l'interface au fichier. Normalement, le fichier doit contenir seulement ceci :

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

Ajouter la 2e interface réseau et activer le DHCP.

## Connexion à Alpine
```plaintext
login: root
```

Pas besoin de renseigner un mot de passe, cela nous connecte directement en root.

Ensuite, pour initialiser la VM Alpine, faire :
```bash
setup-alpine
```

Si Alpine ne trouve pas le répertoire ou si une erreur `wget: bad address` apparaît, ajouter manuellement le répertoire :
```plaintext
http://dl-cdn.alpinelinux.org/alpine/
```

Pour redémarrer sur Alpine :
```bash
reboot
```

## Problèmes Potentiels avec eduroam
Potentiellement, eduroam bloque la VM Alpine, donc passer sur la connexion eduspot.

## Installation et Configuration de DHCPD sur Alpine
```bash
apk add dhcpd
rc-update add dhcpd default
service dhcpd start
```
