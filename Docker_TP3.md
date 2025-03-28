# TP3
# Sur la VM du lab

## Se connecter à distance

```bash
sudo apt update
sudo apt install openssh-server
ip a
```

## Pour installer Docker

```bash
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl status docker
```

## Tester l'installation

```bash
sudo docker run hello-world
```

## Pour éviter de taper sudo à chaque fois

```bash
sudo usermod -aG docker $USER
sudo reboot
```

## Conteneur interactif Alpine

```bash
docker pull alpine
docker run -it --name alp1 alpine sh
```

### Sur la Alpine

```bash
df -h # Espace disque
free -m # Mémoire
ip a # Réseau
touch /tmp/toto.txt
exit
```

### Sur la machine hôte

```bash
docker ps -a | grep alp1
# a723a31ea7b8   alpine        "sh"       9 minutes ago    Up 9 minutes                          alp1
```

## Conteneur en mode démon

### Lancer le conteneur

```bash
docker run -d --name alp2 alpine tail -f /dev/null
```

### Récupérer les paramètres

```bash
docker inspect alp2    # Détails complets
docker stats alp2      # Utilisation des ressources
```

### Tester l'existence de toto.txt

```bash
docker exec alp2 ls /tmp  # Le fichier n'existe pas (conteneur isolé)
```

### Créer titi.txt depuis l'hôte

```bash
docker exec alp2 touch /tmp/titi.txt
```

### Arrêter le conteneur

```bash
docker stop alp2
```

### Lister les conteneurs

```bash
docker ps -a
```

### Relancer les conteneurs

```bash
docker start alp1 alp2
```

### Récupérer les paramètres

```bash
docker stats alp1 alp2
```

### Vérifier les fichiers

```bash
docker exec alp1 ls /tmp # toto.txt normalement il est présent
docker exec alp2 ls /tmp # titi.txt normalement il est présent
```

### Arrêter les conteneurs

```bash
docker stop alp1 alp2
```

### Supprimer les conteneurs

```bash
docker rm alp1 alp2
```

## Conteneur httpd simple

### Télécharger l'image

```bash
docker pull httpd
```

### Lancer le conteneur avec suppression automatique

```bash
docker run -d --name web1 --rm httpd
```

### Récupérer l'IP et les processus

```bash
docker inspect web1 | grep IPAddress # Adresse IP
docker top web1 # Processus
```

### Arrêter le conteneur

```bash
docker stop web1
```

## Forward de port

### Lancer le conteneur

```bash
docker run -it --name web2 -p 8080:80 --rm httpd
```

### Accéder depuis l'hôte

```bash
curl http://localhost:8080
# ou via un navigateur : http://localhost:8080
# ou dans notre cas mettre l'adresse IP http://<IP_UBUNTU>:8080
```

### Arrêter le conteneur

```bash
docker stop web2
```

## Partage de répertoire

### Créer un répertoire et un fichier HTML

```bash
mkdir ~/my_web
echo "<h1>Hello Docker !</h1>" > ~/my_web/index.html
```

### Lancer le conteneur

```bash
docker run -d --name web3 -p 80:80 -v ~/my_web:/usr/local/apache2/htdocs httpd
```

### Accéder au serveur

```bash
curl http://localhost
# ou via navigateur : http://localhost
# ou dans notre cas mettre l'adresse IP http://<IP_UBUNTU>:80
```

### Arrêter le conteneur

```bash
docker stop web3
```
