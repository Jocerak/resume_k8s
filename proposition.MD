Voici le texte que vous avez fourni, remis en forme et clarifié pour une meilleure lisibilité.

---

# Sujet : Je suis un étudiant qui manque de budget

## CHAPITRE 1

### **Solution**

#### 1. Mettre la solution en local (installer sur votre PC)

#### 2. Virtualiser une machine Linux sur votre PC physique Windows

#### 3. Installer tous les prérequis

- **Installer Minikube sur votre VM** : Vous pouvez suivre le lien du site officiel en cliquant [ici](https://minikube.sigs.k8s.io/docs/start).

**Minikube**, c'est quoi ?
> Minikube est un Kubernetes local, axé sur la facilité d'apprentissage et de développement pour Kubernetes. Tout ce dont vous avez besoin, c'est d'un conteneur Docker (ou similaire compatible) ou d'un environnement de machine virtuelle.

- **Vérification de l'installation** : Pour vérifier si l'installation est bien faite, exécutez la commande suivante : `minikube --help`

- [Documentation pour l'installation de kubectl](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/)

Pour installer **kubectl**, collez la commande suivante dans votre terminal :

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl 
```

Pour vérifier l'installation de **kubectl**, exécutez la commande :  
`kubectl --help`

#### 4. Créer un cluster avec Minikube

Pour créer un cluster Kubernetes, tapez la commande suivante dans votre terminal :  
`minikube start`

Un cluster est un réseau de machines qui communiquent entre elles pour faire fonctionner vos applications. Voici un exemple de retour après l'exécution de `minikube start` :

>😄  minikube v1.34.0 sur Ubuntu 24.04
>✨  Choix automatique du pilote docker
>📌  Utilisation du pilote Docker avec le privilège root
>👍  Démarrage du nœud "minikube" primary control-plane dans le cluster "minikube"
>🚜  Extraction de l'image de base v0.0.45...
>🔥  Création de docker container (CPU=2, Memory=3900Mo) ...
>🐳  Préparation de Kubernetes v1.31.0 sur Docker 27.2.0...
>    ▪ Génération des certificats et des clés
>    ▪ Démarrage du plan de contrôle ...
>    ▪ Configuration des règles RBAC ...
>🔗  Configuration de bridge CNI (Container Networking Interface)...
>🔎  Vérification des composants Kubernetes...
>    ▪ Utilisation de l'image gcr.io/k8s-minikube/storage-provisioner:v5
>🌟  Modules activés: default-storageclass, storage-provisioner
>🏄  Terminé ! kubectl est maintenant configuré pour utiliser "minikube" cluster et espace de noms "default" par défaut.

---

### Déploiement en lignes de commande

#### Répartir les déploiements sur deux machines (nodes)

Avant de commencer, supprimez le cluster sur l'ancien déploiement en exécutant :  
`minikube delete`

Pour recréer le cluster avec deux machines, tapez :  
`minikube start --nodes 2`

Pour afficher le statut des deux nodes, exécutez :  
`minikube status`

#### Déployer une image sur le cluster

Pour créer un déploiement avec deux réplicas, exécutez la commande suivante :

```bash
kubectl create deployment myserver --replicas 2 --image=kicbase/echo-server:1.0
```

Explication de la commande :
- `kubectl create deployment myserver` : Crée un déploiement nommé `myserver`.
- `--replicas 2` : Crée 2 réplicas (machines).

Pour vérifier l'état du déploiement, exécutez les commandes suivantes :
- `kubectl get pods` : Affiche les pods.
- `kubectl get deployment` : Affiche les déploiements.
- `kubectl get services` : Affiche les services.
- `kubectl get pods -o wide` : Affiche le statut détaillé des pods.

Exemple de retour pour les pods :

| NAME                          | READY | STATUS  | RESTARTS | AGE  | IP           | NODE      |
|-------------------------------|-------|---------|----------|------|--------------|-----------|
| hello-matrix-7d864756b7-4d5r9 | 1/1   | Running | 0        | 17m  | 10.244.0.3   | minikube  |

---

### Déploiement par fichier de configuration

Vous pouvez aussi définir la performance de votre déploiement, par exemple en utilisant 4 CPUs et 4096 Mo de mémoire :

```bash
minikube start --cpus 4 --memory 4096
```

#### Déployer une image Docker dans un cluster Kubernetes

##### Déployer une application

Exécutez la commande suivante pour créer un déploiement avec l'image `fredericeducentre/matrix` :

```bash
kubectl create deployment hello-matrix --image=fredericeducentre/matrix
```

##### Exposer l'application

Pour exposer le déploiement au port 80, exécutez :

```bash
kubectl expose deployment hello-matrix --type=NodePort --port=80
```

##### Vérifier les déploiements et les pods

Pour voir les déploiements et pods, utilisez :

```bash
kubectl get deployment
kubectl get pods
```

##### Voir les services

Exécutez cette commande pour afficher les services exposés :

```bash
kubectl get services
```

Pour accéder au service via Minikube, utilisez :

```bash
minikube service hello-matrix
```

---

# CHAPITRE 2

## Déploiement de deux services qui doivent fonctionner ensemble (Application et Base de données)

Exemple : Déploiement de MySQL et PostgreSQL.

### 1. Prérequis

Avant de déployer les fichiers YAML, assurez-vous que :
- Vous avez un cluster Kubernetes fonctionnel.
- Vous avez `kubectl` configuré pour interagir avec votre cluster.
- Les images Docker (MySQL et WordPress) sont disponibles dans le registre Docker Hub ou un autre registre accessible par Kubernetes.

### 2. Déployer la première partie : MySQL (Base de données pour WordPress)

#### Étape 2.1 : Appliquer le fichier YAML

Sauvegardez le contenu du fichier YAML dans un fichier, par exemple `wordpress-mysql.yaml`, puis exécutez :

```bash
kubectl apply -f wordpress-mysql.yaml
```

Cela va créer :
- Un **Service** exposant le port 3306 pour MySQL.
- Un **Deployment** pour déployer MySQL avec des variables d'environnement telles que `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, et `MYSQL_PASSWORD`.

#### Étape 2.2 : Vérifier le déploiement de MySQL

Exécutez les commandes suivantes pour vérifier si le déploiement est réussi :

```bash
kubectl get deployments
kubectl get pods
```

### 3. Déployer la deuxième partie : WordPress (Frontend)

#### Étape 3.1 : Appliquer le fichier YAML

Sauvegardez le contenu du fichier YAML dans un fichier, par exemple `wordpress.yaml`, puis exécutez :

```bash
kubectl apply -f wordpress.yaml
```

Cela va créer :
- Un **Service** exposant WordPress sur le port 80 avec un `NodePort`.
- Un **Deployment** pour déployer WordPress avec des variables d'environnement pointant vers le service MySQL (`wordpress-mysql`).

#### Étape 3.2 : Vérifier le déploiement de WordPress

Exécutez les commandes suivantes pour vérifier si le déploiement est réussi :

```bash
kubectl get deployments
kubectl get pods
```

### 4. Accéder à WordPress

Étant donné que le service WordPress est de type `NodePort`, vous pouvez y accéder via l'adresse IP du nœud et le port attribué.

Exécutez cette commande pour obtenir le port exposé :

```bash
kubectl get services wordpress
```

### 5. Vérifications finales

- Vérifiez les pods en cours d'exécution avec :  
  ```bash
  kubectl get pods
  ```
  
- Vérifiez les services exposés avec :  
  ```bash
  kubectl get services
  ```

- Si nécessaire, consultez les logs des pods avec :  
  ```bash
  kubectl logs <pod-name>
  ```

### Résumé des commandes principales :
1. Appliquer le fichier de déploiement pour MySQL :
   ```bash
   kubectl apply -f wordpress-mysql.yaml
   ```

2. Appliquer le fichier de déploiement pour WordPress :
   ```bash
   kubectl apply -f wordpress.yaml
   ```

3. Vérifier les ressources déployées :
   ```bash
   kubectl get deployments
   kubectl get pods
   kubectl get services
   ```