# TP6 : Ansible - Commandes et Explications

## 1. Configuration initiale

### Définir le hostname des machines
```bash
sudo hostnamectl set-hostname ansible   # Sur la machine Serveur Ansible
sudo hostnamectl set-hostname virtu     # Sur la machine Virtualisation
```
**Explication** : Configure le nom d'hôte permanent des machines.

## 2. Configuration SSH

### Copier les clés SSH vers les serveurs
```bash
ssh-copy-id -i ~/.ssh/tpinfo0602.pub ubuntu@ansible
ssh-copy-id -i ~/.ssh/tpinfo0602.pub ubuntu@virtu
```
**Explication** : Autorise la connexion SSH sans mot de passe via clé publique.

### Générer une clé RSA sur ansible pour virtu
```bash
ssh ansible
ssh-keygen -t rsa                   # Génère une paire de clés
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@virtu
```
**Explication** : Permet à ansible de se connecter à virtu sans mot de passe.

## 3. Installation des logiciels

### Installer Ansible sur ansible
```bash
sudo apt update && sudo apt install ansible -y
```

### Installer Docker sur virtu
```bash
sudo apt update && sudo apt install docker.io -y
```

## 4. Gestion de l'inventaire

### Fichier `inventory.ini`
```ini
[local]
ansible ansible_connection=local

[remote]
virtu ansible_host=10.11.50.194 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/ansible
virtu2 ansible_host=10.11.50.194 ansible_user=user
```

## 5. Commandes ad hoc

### Tester la connexion
```bash
ansible all -i inventory.ini -m ping
```

### Récupérer des informations réseau
```bash
ansible virtu -i inventory.ini -m command -a "ip a"
```

### Comparer les processeurs
```bash
ansible ansible -i inventory.ini -m command -a "lscpu"
ansible virtu -i inventory.ini -m command -a "lscpu"
```

## 6. Gestion des utilisateurs

### Créer un utilisateur `user` sur virtu
```bash
ssh virtu
sudo adduser user    # Suivre les prompts
```

### Mettre à jour l'inventaire pour `user`
```ini
virtu ansible_host=10.11.50.YY ansible_user=user
```

## 7. Script `getstate.sh`

### Créer le script sur ansible
```bash
#!/bin/bash
echo "$(date)" > /home/user/state.txt
free -h >> /home/user/state.txt
lscpu >> /home/user/state.txt
```

### Copier/exécuter le script
```bash
ansible virtu -i inventory.ini -m copy -a "src=getstate.sh dest=/home/user/getstate.sh" -k --become -K
ansible virtu -i inventory.ini -m shell -a "/home/user/getstate.sh" -k --become -K
```

**Flags** :
- `-k` : Demande le mot de passe SSH.
- `-K` : Demande le mot de passe sudo.
- `--become` : Active les privilèges sudo.

## 8. Gestion des erreurs courantes

### Erreur "Permission denied"
```bash
# Forcer les permissions avec sudo
ansible virtu -i inventory.ini -m file -a "path=/home/user/getstate.sh state=absent" -k --become -K
```

### Remplacer `command` par `shell` pour les opérateurs
```bash
ansible virtu -i inventory.ini -m shell -a "chmod +x /home/user/getstate.sh && /home/user/getstate.sh" -k -K
```

## 9. Playbook Ansible

### Fichier `deploy_script.yml`
```yaml
- hosts: virtu
  become: yes
  tasks:
    - name: Créer /home/user
      file:
        path: /home/user
        state: directory
        owner: user
        mode: "0755"

    - name: Copier le script
      copy:
        src: getstate.sh
        dest: /home/user/getstate.sh
        mode: "0744"

    - name: Exécuter le script
      shell: "/home/user/getstate.sh"
```

### Lancer le playbook
```bash
ansible-playbook -i inventory.ini deploy_script.yml -k -K
```

## 10. Vérifications finales

### Récupérer le fichier `state.txt`
```bash
ansible virtu -i inventory.ini -m fetch -a "src=/home/user/state.txt dest=/home/ubuntu/ flat=yes" -k
```

### Afficher le résultat
```bash
cat /home/ubuntu/state.txt
```

### Résumé des flags
| Flag       | Description                                   |
|------------|-----------------------------------------------|
| `-i`       | Spécifie le fichier d'inventaire             |
| `-m`       | Module Ansible à utiliser (ex: copy, shell)  |
| `-a`       | Arguments pour le module                     |
| `-k`       | Demande le mot de passe SSH                  |
| `-K`       | Demande le mot de passe sudo                 |
| `--become` | Active l'escalade de privilèges (sudo)       |

---

## **1. Premier Playbook : Copie et exécution de script**

### **Fichier : `question3.yml`**
```yaml
- name: "Copier et exécuter le script getstate.sh"
  hosts: virtu
  become: yes # Activation des privilèges sudo
  tasks:
    - name: "Créer le répertoire /home/user"
      ansible.builtin.file:
        path: /home/user
        state: directory
        owner: user
        group: user
        mode: "0755"

    - name: "Copier le script getstate.sh"
      ansible.builtin.copy:
        src: getstate.sh
        dest: /home/user/getstate.sh
        owner: user
        mode: "0744"

    - name: "Exécuter le script"
      ansible.builtin.shell: "/home/user/getstate.sh"

    - name: "Récupérer le fichier state.txt"
      ansible.builtin.fetch:
        src: /home/user/state.txt
        dest: ./state.txt
        flat: yes

    - name: "Supprimer le script"
      ansible.builtin.file:
         path: /home/user/getstate.sh
         state: absent
```

### Exécution
```bash
ansible-playbook -i inventory.ini copy_execute_playbook.yml -k -K
cat state.txt
```

---

## **2. Playbook : Déployer un conteneur Apache**

### **Fichier : `docker_apache_playbook.yml`**
```yaml
- name: "Déployer un conteneur Apache"
  hosts: virtu
  become: yes
  tasks:
   - name: "Mettre à jour les paquets APT"
     ansible.builtin.apt:
       update_cache: yes

   - name: "Installer Docker"
     ansible.builtin.apt:
       name: docker.io
       state: present

   - name: "Télécharger l'image httpd"
     ansible.builtin.docker_image:
        name: httpd:latest
        source: pull

   - name: "Lancer le conteneur Apache"
     ansible.builtin.docker_container:
        name: my_apache
        image: httpd:latest
        state: started
        ports:
          - "8080:80"
```

### Exécution
```bash
ansible-playbook -i inventory.ini docker_apache_playbook.yml -k -K
curl http://<IP_VIRTU>:8080
```

---

## **3. Playbook : Déployer Apache avec partage de volume**

### **Fichier : `docker_apache_volume_playbook.yml`**
```yaml
- name: "Déployer Apache avec partage de volume"
  hosts: virtu
  become: yes
  tasks:
    - name: "Créer le répertoire partagé"
      ansible.builtin.file:
        path: /home/user/apache_volume
        state: directory
        mode: "0755"

    - name: "Copier index.html dans le volume"
      ansible.builtin.copy:
        src: index.html
        dest: /home/user/apache_volume/index.html

    - name: "Lancer le conteneur avec volume partagé"
      ansible.builtin.docker_container:
        name: my_apache
        image: httpd:latest
        state: started
        ports:
          - "8000:80"
        volumes:
          - "/home/user/apache_volume:/usr/local/apache2/htdocs"
```

### Exécution
```bash
ansible-playbook -i inventory.ini docker_apache_volume_playbook.yml -k -K
curl http://<IP_VIRTU>:8000
```
