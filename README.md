# OpenstackNxtCloud


source PCP-G43FMLT-openrc.sh
cd ~/.ssh/
ssh-keygen
openstack server create --flavor a4-ram16-disk80-perf1 --image 1a1a2fe9-e9b7-4fbf-b34d-c6cd71e1e8ab --network ext-net1 --key-name SshKeyRayan nextcloud-vm_Rayan
ssh ubuntu@<IP_PUBLIQUE>

1\. **Installation et préparation du système**
----------------------------------------------

bash

`
sudo  apt update
sudo apt upgrade -y
sudo  apt  install docker.io nginx certbot python3-certbot-nginx -y
sudo systemctl enable --now docker  `

- **But** : Installer Docker, Nginx, Certbot et mettre à jour le système.

* * * * *

2\. **Téléchargement et lancement du conteneur Nextcloud**
----------------------------------------------------------

bash

`sudo  docker pull nextcloud sudo  docker run -d -p 8080:80 nextcloud `

-   **But** : Télécharger puis lancer Nextcloud en mode test.

* * * * *

3\. **Configuration du reverse proxy Nginx**
--------------------------------------------

bash

`
sudo  nano /etc/nginx/sites-available/nextcloud 
sudo  ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
sudo nginx -t sudo systemctl reload nginx `

-   **But** : Créer la configuration Nginx pour faire reverse proxy vers Nextcloud.

* * * * *

4\. **Obtention du certificat HTTPS**
-------------------------------------

bash

`sudo certbot --nginx -d rayab.argb.cfpt.info `

-   **But** : Sécuriser l'accès à Nextcloud avec HTTPS via Let's Encrypt.

* * * * *

5\. **Gestion des volumes persistants (Cinder)**
------------------------------------------------

bash

`sudo  docker run -d \   -p 8080:80 \  
-v /mnt/cinder/nextcloud-data:/var/www/html/data \ 
-v /mnt/cinder/nextcloud-config:/var/www/html/config \   
--name nextcloud \
nextcloud `

-   **But** : Lancer Nextcloud en Docker avec persistance des données et de la config sur le volume Cinder.

* * * * *

6\. **Vérification de la persistance**
--------------------------------------

bash

`sudo  docker  exec nextcloud ls -l /var/www/html/data 
 sudo  docker  exec nextcloud ls -l /var/www/html/config `

-   **But** : Vérifier que les volumes sont bien montés et utilisés par Nextcloud.

* * * * *

7\. **Modification de la configuration Nextcloud**
--------------------------------------------------

bash

`sudo  nano /mnt/cinder/nextcloud-config/config.php `

-   **But** : Ajouter les trusted domains et le protocole HTTPS dans la config Nextcloud.

* * * * *

8\. **Création du script de sauvegarde**
----------------------------------------

bash

`sudo  nano /mnt/cinder/nextcloud-data/backup-nextcloud.sh
 sudo  chmod +x /mnt/cinder/nextcloud-data/backup-nextcloud.sh `

-   **But** : Créer et rendre exécutable le script de sauvegarde automatique.

* * * * *

9\. **Automatisation de la sauvegarde**
---------------------------------------

bash

`sudo  crontab -e `

-   **But** : Ajouter la sauvegarde quotidienne dans la crontab root.

* * * * *

10\. **Lancement manuel et vérification des sauvegardes**
---------------------------------------------------------

bash

`sudo /mnt/cinder/nextcloud-data/backup-nextcloud.sh 
 ls /mnt/cinder/nextcloud-backup/`
