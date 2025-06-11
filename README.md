# 🚀 Déploiement Nextcloud sur OpenStack avec Docker, Nginx, Certbot & Volumes Cinder

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


🏆 BONUS 1 : Auto-scaling Nextcloud avec Heat + Script (OpenStack)

Ajouter l’auto-scaling à ton infrastructure VM grâce à OpenStack Heat :

text
# autoscale-nextcloud.yaml
heat_template_version: 2016-10-14
description: Auto-scaling Nextcloud avec Heat

parameters:
  image:
    type: string
    default: "1a1a2fe9-e9b7-4fbf-b34d-c6cd71e1e8ab"
  flavor:
    type: string
    default: "a4-ram16-disk80-perf1"
  network:
    type: string
    default: "ext-net1"
  key_name:
    type: string
    default: "SshKeyRayan"

resources:
  nextcloud_group:
    type: OS::Heat::AutoScalingGroup
    properties:
      cooldown: 120
      desired_capacity: 1
      min_size: 1
      max_size: 3
      resource:
        type: OS::Nova::Server
        properties:
          image: { get_param: image }
          flavor: { get_param: flavor }
          networks: [{ network: { get_param: network } }]
          key_name: { get_param: key_name }

  scaleup_policy:
    type: OS::Heat::ScalingPolicy
    properties:
      auto_scaling_group_id: { get_resource: nextcloud_group }
      adjustment_type: change_in_capacity
      scaling_adjustment: 1
      cooldown: 120

  scaledown_policy:
    type: OS::Heat::ScalingPolicy
    properties:
      auto_scaling_group_id: { get_resource: nextcloud_group }
      adjustment_type: change_in_capacity
      scaling_adjustment: -1
      cooldown: 120

  cpu_alarm_high:
    type: OS::Aodh::GnocchiAggregationByResourcesAlarm
    properties:
      metric: cpu
      aggregation_method: rate:mean
      granularity: 60
      evaluation_periods: 1
      threshold: 80
      resource_type: instance
      comparison_operator: gt
      alarm_actions: [{ get_attr: [scaleup_policy, alarm_url] }]

  cpu_alarm_low:
    type: OS::Aodh::GnocchiAggregationByResourcesAlarm
    properties:
      metric: cpu
      aggregation_method: rate:mean
      granularity: 60
      evaluation_periods: 1
      threshold: 20
      resource_type: instance
      comparison_operator: lt
      alarm_actions: [{ get_attr: [scaledown_policy, alarm_url] }]

Déploie le stack :

bash
source PCP-G43FMLT-openrc.sh
openstack stack create -t autoscale-nextcloud.yaml nextcloud-autoscale

🎯 Objectif : Les VM Nextcloud scaleront automatiquement selon la charge CPU !
🏆 BONUS 2 : Auto-scaling Nextcloud avec Kubernetes (HPA)

Si tu déploies Nextcloud sur Kubernetes, utilise le Horizontal Pod Autoscaler :

text
# nextcloud-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
  template:
    metadata:
      labels:
        app: nextcloud
    spec:
      containers:
      - name: nextcloud
        image: nextcloud
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
          limits:
            cpu: "1"
            memory: "2Gi"

Active le HPA :

bash
kubectl autoscale deployment nextcloud --cpu-percent=70 --min=1 --max=5
kubectl get hpa

🎯 Objectif : Le nombre de pods Nextcloud s’ajuste automatiquement selon la charge !
