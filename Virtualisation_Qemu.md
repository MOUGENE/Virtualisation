# VM Ubuntu

```bash
sudo apt update
sudo apt install openssh-server
```

On active le VPN et on se connecte en SSH

```bash
sudo apt-get install qemu-system
```

```bash
wget https://filesender.renater.fr/?s=download&token=044eb74e-785e-4af0-a19e-20c9f3945344 -O alp2.img
```

--> Ne fonctionne pas à cause du `&` car il est interprété comme un opérateur

```bash
wget "https://filesender.renater.fr/?s=download&token=044eb74e-785e-4af0-a19e-20c9f3945344" -O alp2.img
```

--> Il faut protéger l'URL  
--> Pas le bon non plus

Il faut faire un clic droit sur le téléchargement pour avoir la bonne adresse de téléchargement

```bash
wget "https://filesender.renater.fr/download.php?token=044eb74e-785e-4af0-a19e-20c9f3945344&files_ids=49581119" -O alp2.img
```

Pour lancer la VM Alpine2 :

```bash
qemu-system-x86_64 -k fr -m 512 -drive file=./alp2.img,format=qcow2 -net nic -net user,hostfwd=tcp::10222-:22,hostfwd=tcp::8080-:80 -display none
```

Ensuite ouvrir un autre terminal et faire :

```bash
curl "adresse_IP_de_la_machine":8080
ssh "nom_machine"@"adresse_IP_de_la_machine"
```

# Partie Alpine3

```plaintext
login : root
mdp : 
```

```bash
apk update
apk add nano
nano /etc/apk/repositories
```

Décommenter la ligne des dépôts Community

```bash
apk add figlet
apk add apache2
echo "<h1>TP2 en cours ! </h1>" > /var/www/localhost/htdocs/index.html
rc-service apache2 start
netstat -tlnp  # Pour vérifier sur quels ports écoute le serveur apache
apk add curl
curl http://localhost:80
```

Pour accéder au site web depuis la machine hôte, il faut se connecter sur la VM Ubuntu en appliquant la redirection de port de la VM Alpine Qemu

En gros :

```plaintext
http://10.11.5.113:8081  # On lance le site grâce à l'IP Ubuntu en redirigeant sur le port 8081 lié à la VM Alpine
```

Pour relancer la VM Alpine après, il faut enlever les options :

```plaintext
-boot d
-cdrom
```

# Partie VM Alpine4 VNC

```bash
qemu-system-x86_64 \
  -m 1G \
  -drive file=./alp4.qcow2,format=qcow2 \
  -cdrom alpine-virt-3.21.0-x86_64.iso \
  -vnc :0 \  # Permet de lancer un environnement VNC
  -k fr
```

Ensuite on va sur VNC et on se connecte grâce à l'IP de la machine Ubuntu :

```plaintext
10.11.5.113
```

Au lieu de rebooter la VM :

il faut retirer l'option :

```plaintext
-cdrom
```

Vérifier la taille d'installation et préciser plus d'espace lors de la création de la VM.

Lors de l'installation, il faut choisir "y" lors de l'écriture sur le disque "sda".
