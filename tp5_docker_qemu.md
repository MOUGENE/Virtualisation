# TP5
# Résumé des commandes du TP Docker & Qemu

---

## **Question 1 : Configuration et Installation**

### Nommage des machines
```bash
sudo hostnamectl set-hostname machine1  # Sur machine1
sudo hostnamectl set-hostname machine2  # Sur machine2
```

### Configuration réseau statique (Netplan)
```yaml
# /etc/netplan/01-config.yaml (machine1)
network:
  version: 2
  ethernets:
    eth0:
      addresses: [10.11.49.50/28]
      routes:
        - to: default
          via: 10.11.49.62
      nameservers:
        addresses: [8.8.8.8]
```

### Clés SSH
```bash
ssh-keygen -t rsa -f ~/.ssh/tpinfo0602  # Génération des clés
ssh-copy-id -i ~/.ssh/tpinfo0602.pub ubuntu@machine1  # Copie sur les machines
```

### Installation Docker et Qemu
```bash
sudo apt install docker.io qemu-system -y
sudo systemctl enable --now docker
```

---

## **Question 2 : Portainer**

### Lancement de Portainer
```bash
docker run -d -p 9000:9000 --name portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  portainer/portainer
```

### Ajout de machine2 à Portainer
- **URL dans Portainer** : `tcp://10.11.49.51:2375`

**Commande pour activer l'API Docker sur machine2 :**
```bash
sudo nano /lib/systemd/system/docker.service  # Ajouter "-H tcp://0.0.0.0:2375"
sudo systemctl daemon-reload && sudo systemctl restart docker
```

---

## **Question 3 : Qemu et Docker**

### Copie de l'image Qemu
```bash
scp -i ~/.ssh/tpinfo0602 alp2.img ubuntu@machine2:~
```

### Démarrage de la VM Qemu
```bash
qemu-system-x86_64 -hda alp2.img -m 512 -net user,hostfwd=tcp::2222-:22 -net nic -nographic
```

### Accès au conteneur Apache depuis Qemu
```bash
curl http://10.11.49.50:8080  # Dans la VM Qemu
```

---

## **Question 4 : Urbanisation**

### Docker Compose pour machine1 (Apache/PHP)
```yaml
# docker-compose.yml
services:
  web:
    image: php:apache
    ports:
      - "80:80"
    environment:
      - DB_HOST=machine2
```

### Docker Compose pour machine2 (MySQL)
```yaml
# docker-compose.yml
services:
  db:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=secret
    ports:
      - "3306:3306"
```

### Test de connexion MySQL
```bash
docker exec -it web bash
apt update && apt install -y default-mysql-client  # Installation du client
mysql -h machine2 -u root -p  # Mot de passe : secret
```

---

## **Commandes utiles pour résoudre les erreurs**

### Permission Docker
```bash
sudo usermod -aG docker $USER && newgrp docker
```

### Réinstaller Portainer
```bash
docker stop portainer && docker rm portainer
docker volume rm portainer_data
docker run -d -p 9000:9000 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

### Installer Docker Compose
```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

---

## **Tips :**
- Pour relancer Docker Compose après modification : 
  ```bash
  docker-compose up -d --force-recreate
  ```
- Utilisez `docker-compose logs` pour déboguer.
