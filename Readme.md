# PROJET DEVOPS - Orchestration : Déploiement Odoo, pgAdmin PostgreSQL:     ESGI - Indrit KULLA 5SRC2

## Contexte

Ce projet consiste à déployer sur un cluster Kubernetes les services suivants pour la société IC GROUP :

- L'ERP Odoo
- 	Un site vitrine développé par l'équipe IC GROUG
- 	L'interface de gestion pgAdmin pour PostgreSQL


L'entreprise IC GROUP souhaite exposer deux applications internes à travers un site web vitrine :
- 	Odoo 13.0 Community Edition
- 	pgAdmin 4

Liens utiles : 

- Site officiel :[ https://www.odoo.com/ ](https://www.odoo.com/) 
- GitHub officiel:[ https://github.com/odoo/odoo.git ](https://github.com/odoo/odoo.git) 
- Docker Hub officiel :[ https://hub.docker.com/_/odoo ](https://hub.docker.com/_/odoo) 

## Pré-requis

Les outils suivants doivent être installés sur la VM
- 	Git
- 	Docker
- 	Kubernetes (K3S)
- 	Un cluster Kubernetes fonctionnel
- 	kubectl
- 	Les fichiers YAML de déploiement (présents dans ce dépôt)


## I- Étapes de déploiement

### 1. Conteneurisation de l'application Flask

Un fichier Dockerfile est écrit pour conteneuriser l'application Flask

```
FROM python:3.6-alpine
WORKDIR /opt
COPY . /opt
RUN pip install flask==1.1.2
ENV ODOO_URL=https://www.odoo.com
ENV PGADMIN_URL=https://www.pgadmin.org
EXPOSE 8080
ENTRYPOINT ["python", "app.py"]
```

### 2. Build de l'image

```
docker build -t ic-webapp:1.0 .
```

![image](https://github.com/user-attachments/assets/b11fec73-f1e1-48b0-80e1-be06f1abb9ab)

### 3. Test local avec variables d'environnement

```
docker run -d --name test-ic-webapp -p 8080:8080 -e ODOO_URL=https://www.odoo.com -e PGADMIN_URL=https://www.pgadmin.org ic-webapp:1.0.
```
C'est ok ✅

![image](https://github.com/user-attachments/assets/45b94279-3b73-430b-907a-65fcad2582bf)


### 4. Push vers Docker Hub

```
docker tag ic-webapp:1.0 titan111/ic-webapp:1.0
docker push  titan111/ic-webapp:1.0
```

![image](https://github.com/user-attachments/assets/bcaf27ab-662e-4b6f-ba1a-8115be4535d9)

![image](https://github.com/user-attachments/assets/95ce44a3-7110-4794-9a45-72b933c7e50d)


### 5. Création du namespace

```
kubectl create namespace icgroup
```
Cela permet d'isoler tous les composants du projet dans un namespace dédié

### 6. Déploiement des ressources

```
# Volume persistant pour PostgreSQL
kubectl apply -f postgres-pvc.yaml -n icgroup

# Déploiement de PostgreSQL
kubectl apply -f postgres-deployment.yaml -n icgroup

# Volume persistant pour Odoo
kubectl apply -f odoo-volume.yaml -n icgroup

# Déploiement d’Odoo
kubectl apply -f odoo-deployment.yaml -n icgroup

# Déploiement de pgAdmin
kubectl apply -f pgadmin-deployment.yaml -n icgroup

# Déploiement de l’application web
kubectl apply -f ic-webapp-deployment.yaml -n icgroup

# Déploiement des services
kubectl apply -f services.yaml -n icgroup
```

### 6. Configuration de pgAdmin

Créez un ConfigMap avec le fichier servers.json qui est fourni pour pgAdmin

```
kubectl create configmap pgadmin-config --from-file=servers.json -n icgroup
```

### 7. Vérification des déploiements

```
kubectl get all -n icgroup
```

![image](https://github.com/user-attachments/assets/008fc6d0-4a5c-467d-84e6-06efbfa78b19)


### 8. Accès aux applications

Pour accéder à pgAdmin ou Odoo depuis votre machine locale il faut faire des tunnels SSH avec les ports correspondent: 

### Odoo

```
ssh -L 8069:10.43.103.236:8069 -L 8069:10.43.103.236:8069 linus1@192.168.136.162
```
 ![image](https://github.com/user-attachments/assets/b4777b57-457e-473f-a806-b2493df7b157)

### pgAdmin

```
ssh -L 80:10.43.106.111:80 -L 80:10.43.106.111:80 linus1@192.168.136.162
```

![image](https://github.com/user-attachments/assets/6f840011-1abb-476d-adf2-85203689a9fe)

### Site de Vitrine

```
ssh -L 8080:10.43.78.22:8080 -L 8080:10.43.78.22:8080 linus1@192.168.136.16
```

![image](https://github.com/user-attachments/assets/f2071bed-9f51-4507-a0e9-4d4e5f348dcd)

## Rapport final ✅

Toutes les applications sont fonctionnelles, accessibles via le site vitrine

![image](https://github.com/user-attachments/assets/eeb7aa9d-bd1e-45d4-892c-e6b7230170fb)

Réalisé par :

Indrit KULLA 5SRC2

