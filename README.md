🚀 Déploiement Nextcloud sur OpenStack avec Docker, Nginx, Certbot & Volumes Cinder

Voici un guide ultra-visuel et structuré pour briller lors de l’installation de Nextcloud sur une VM OpenStack, avec Docker, Nginx, Certbot et gestion de la persistance via Cinder.
1. Préparation de la VM et installation des outils

bash
sudo apt update
sudo apt upgrade -y
sudo apt install docker.io nginx certbot python3-certbot-nginx -y
sudo systemctl enable --now docker

    Objectif : Système à jour, Docker, Nginx et Certbot prêts à l’emploi.

2. Téléchargement et lancement du conteneur Nextcloud

bash
sudo docker pull nextcloud
sudo docker run -d -p 8080:80 --name nextcloud-test nextcloud

    Objectif : Lancer Nextcloud rapidement en mode test.

3. Configuration du reverse proxy Nginx

bash
sudo nano /etc/nginx/sites-available/nextcloud
# (Ajouter la conf reverse proxy)
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

    Objectif : Rediriger le trafic web vers Nextcloud (port 8080).

4. Obtention d’un certificat HTTPS avec Certbot

bash
sudo certbot --nginx -d rayab.argb.cfpt.info

    Objectif : Sécuriser l’accès à Nextcloud avec HTTPS.

5. Gestion de la persistance avec Cinder (volumes OpenStack)

bash
sudo docker run -d \
  -p 8080:80 \
  -v /mnt/cinder/nextcloud-data:/var/www/html/data \
  -v /mnt/cinder/nextcloud-config:/var/www/html/config \
  --name nextcloud \
  nextcloud

    Objectif : Les données et la config Nextcloud sont stockées de façon persistante sur les volumes Cinder.

6. Vérification du montage des volumes

bash
sudo docker exec nextcloud ls -l /var/www/html/data
sudo docker exec nextcloud ls -l /var/www/html/config

    Objectif : S’assurer que les volumes sont bien utilisés par Nextcloud.

7. Modification de la configuration Nextcloud

bash
sudo nano /mnt/cinder/nextcloud-config/config.php
# Ajouter les trusted domains et forcer HTTPS

    Objectif : Sécuriser la configuration (trusted domains, HTTPS).

8. Création d’un script de sauvegarde automatique

bash
sudo nano /mnt/cinder/nextcloud-data/backup-nextcloud.sh
sudo chmod +x /mnt/cinder/nextcloud-data/backup-nextcloud.sh

    Objectif : Automatiser la sauvegarde de Nextcloud.

9. Automatisation via crontab

bash
sudo crontab -e
# Ajouter : 0 3 * * * /mnt/cinder/nextcloud-data/backup-nextcloud.sh

    Objectif : Sauvegarde quotidienne automatique.

10. Lancement manuel et vérification des sauvegardes

bash
sudo /mnt/cinder/nextcloud-data/backup-nextcloud.sh
ls /mnt/cinder/nextcloud-backup/

    Objectif : Vérifier que les backups sont bien créés.
