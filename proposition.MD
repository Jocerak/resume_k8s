## sujet : Je suis un étudiant qui manque de budget


# CHAPITRE 1
<ins>**solution**
1. Mettre la solution en local (installer sur votre PC) 
2. Virtualiser une machine linux sur votre PC physique windows,
3. installer tout les prerecquis 
    - installer Minikube sur votre VM vous pouvez suivre le lien du site officiel clique [ici](https://minikube.sigs.k8s.io/docs/start)
    
    En premier lieu on va voir c'est quoi minikube 
    > Minikube est un Kubernetes local, axé sur la facilité d'apprentissage et de développement pour Kubernetes.

    Tout ce dont vous avez besoin, c'est d'un conteneur Docker * (ou similaire compatible) ou d'un environnement de machine virtuelle

    - Pour verifier si l'installation et bien faite faire une verifiation en executant les commandes suivant:
        `minikube --help`
    
    - [documentation pour l'installation kubectl](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/)
     Coller cette commande pour votre terminal pour l'intaller 
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl 
```
Pour le verifier executer la commande `kubectl --help`

Etape suivant on va deployer une  machine ou node mais on peux deployer plusieurs machine mais on verra ça après 

Pour creer c'est juste de taper `Minikube start` sur votre terminal il va creer un cluster 
un cluster c'est quoi ? *Cluster ce sont des machines preconfigurer*


*Retour* 

>😄  minikube v1.34.0 sur Ubuntu 24.04
✨  Choix automatique du pilote docker
📌  Utilisation du pilote Docker avec le privilège root
👍  Démarrage du nœud "minikube" primary control-plane dans le cluster "minikube"
🚜  Extraction de l'image de base v0.0.45...
🔥  Création de docker container (CPU=2, Memory=3900Mo) ...
🐳  Préparation de Kubernetes v1.31.0 sur Docker 27.2.0...
    ▪ Génération des certificats et des clés
    ▪ Démarrage du plan de contrôle ...
    ▪ Configuration des règles RBAC ...
🔗  Configuration de bridge CNI (Container Networking Interface)...
🔎  Vérification des composants Kubernetes...
    ▪ Utilisation de l'image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Modules activés: default-storageclass, storage-provisioner
🏄  Terminé ! kubectl est maintenant configuré pour utiliser "minikube" cluster et espace de noms "default" par défaut.

Maintenant on va répartir les déploiements sur deux machines (nodes) 

Avant de commencer :

On supprime le cluster sur l'ancien deployment ; pour le faire execute `minikube delete` sur votre terminal 

Recréer le cluster avec 2 machines

Creer 2 machines (nodes) =>`minikube start --nodes 2`  

Pour afficher le status des 2 nodes =>`minikube status`

Maintenant on va deployer une image sur notre machine 



`kubectl create deployment myserver --replicas 2 --image=kicbase/echo-server:1.0 `
=> explication de cette commande 
kubectl create deployment myserver  => *creer un deployment qui s'appelle myserveur*
`--replicas 2` => *creer 2 machine*
`kubectl get pods`*affiche les pods*
`kubectl get deployment`=> *affiche les deployments*
`kubectl get services` => *affiche les services*

`kubectl get pods -o wide`*affiche le status en version plus détaillé 

*Retour* 
| NAME                          | READY | STATUS  | RESTARTS | AGE  | IP           | NODE      | NOMINATED NODE | READINESS GATES |
|-------------------------------|-------|---------|----------|------|--------------|-----------|----------------|-----------------|
| hello-matrix-7d864756b7-4d5r9 | 1/1   | Running | 0        | 17m  | 10.244.0.3   | minikube  | `<none>`       | `<none>`        |

Toute à l'heure on vu le deployement par ligne de commande 
- Deuxième façon de deployer des machines 
Par un fichier .yaml 
il y a un exemple de ce fichier a votre disposition 

On peut aussi definir aussi la performance de notre deployement 
par exemple on met 4 cpus et 4096 Mo de memoire

`minikube start --cpus 4 memory 4096` 

### Comment deployer une image docker dans un cluster kubernetes 
##### Déployer une application
`kubectl create deployment hello-matrix --image=fredericeducentre/matrix`
=> *deployment.apps/hello-matrix created*

##### Exposer l'application
`kubectl expose deployment hello-matrix --type=NodePort --port=80`
=> *service/hello-matrix exposed*
##### Voir les déploiements et pods
`kubectl get deployment`
| NAME         | READY | UP-TO-DATE | AVAILABLE | AGE   |
|--------------|-------|------------|-----------|-------|
| hello-matrix | 1/1   | 1          | 1         | 3m2s  |

`kubectl get pods`
| NAME                          | READY | STATUS  | RESTARTS | AGE     |
|-------------------------------|-------|---------|----------|---------|
| hello-matrix-7d864756b7-2n9tt  | 1/1   | Running | 0        | 2m47s   |


##### Voir les services
`kubectl get services`
| NAME           | TYPE       | CLUSTER-IP     | EXTERNAL-IP   | PORT(S)        | AGE     |
|---------------|------------|----------------|---------------|----------------|---------|
| hello-matrix  | NodePort   | 10.105.177.234 | <none>        | 80:32119/TCP   | 105s    |
| kubernetes    | ClusterIP  | 10.96.0.1      | <none>        | 443/TCP        | 153m    |

#### Accéder au service via Minikube
`minikube service hello-matrix`

| NAMESPACE |     NAME     | TARGET PORT |            URL            |
|-----------|--------------|-------------|---------------------------|
| default   | hello-matrix |          80 | http://192.168.49.2:32119 |

🎉  Ouverture du service default/hello-matrix dans le navigateur par défaut..




# CHAPITRE 2
## Deployment de deux service qui doivent fonctionner ensemble (application et base de données)

ici on prend comme exemple : on va réalisé un deploiment de Mysql avec Postgres 


Pour déployer ces deux fichiers YAML dans un environnement Kubernetes, vous allez procéder en plusieurs étapes. Je vais expliquer chacune d'elles, de la création des ressources à leur vérification. 

### 1. Prérequis
Avant de déployer les fichiers YAML, assurez-vous que :
- Vous avez un cluster Kubernetes fonctionnel.
- Vous avez `kubectl` configuré pour interagir avec votre cluster.
- Les images Docker (MySQL et WordPress) sont disponibles dans le registre Docker Hub ou un autre registre accessible par Kubernetes.

### 2. Déployer la première partie : MySQL (Base de données pour WordPress)
Le premier fichier YAML crée deux ressources Kubernetes :
- Un **Service** pour exposer MySQL (port 3306).
- Un **Deployment** pour déployer MySQL avec les variables d'environnement nécessaires à sa configuration.

Voici comment procéder pour ce fichier :

#### Étape 2.1 : Appliquer le fichier YAML
1. Sauvegardez le contenu du premier fichier dans un fichier, par exemple `wordpress-mysql.yaml`.
2. Exécutez la commande suivante pour appliquer le fichier YAML :

```bash
kubectl apply -f wordpress-mysql.yaml
```

Cela va créer :
- Un `Service` nommé `wordpress-mysql` exposant le port 3306 pour MySQL.
- Un `Deployment` nommé `wordpress-mysql` qui déploie un conteneur MySQL avec les variables d'environnement comme `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, et `MYSQL_PASSWORD`.

#### Étape 2.2 : Vérifier le déploiement de MySQL
Vérifiez si le déploiement de MySQL est bien effectué en exécutant :

```bash
kubectl get deployments
kubectl get pods
```

Cela devrait afficher un pod pour `wordpress-mysql` en cours d'exécution.

### 3. Déployer la deuxième partie : WordPress (Frontend)
Le deuxième fichier YAML crée également deux ressources Kubernetes :
- Un **Service** pour exposer l'application WordPress via un type `NodePort`.
- Un **Deployment** pour déployer WordPress avec les informations de connexion à la base de données MySQL.

Voici comment procéder pour ce fichier :

#### Étape 3.1 : Appliquer le fichier YAML
1. Sauvegardez le contenu du deuxième fichier dans un fichier, par exemple `wordpress.yaml`.
2. Exécutez la commande suivante pour appliquer le fichier YAML :

```bash
kubectl apply -f wordpress.yaml
```

Cela va créer :
- Un `Service` nommé `wordpress` exposant le port 80 et le rendant accessible via un `NodePort`.
- Un `Deployment` nommé `wordpress` qui déploie un conteneur WordPress avec les variables d'environnement comme `WORDPRESS_DB_HOST`, `WORDPRESS_DB_PASSWORD`, et `WORDPRESS_DB_USER` pointant vers le service MySQL (`wordpress-mysql`).

#### Étape 3.2 : Vérifier le déploiement de WordPress
Vérifiez si le déploiement de WordPress est bien effectué en exécutant :

```bash
kubectl get deployments
kubectl get pods
```

Cela devrait afficher un pod pour `wordpress` en cours d'exécution.

### 4. Accéder à WordPress
Étant donné que le service `wordpress` est de type `NodePort`, vous pouvez accéder à votre application WordPress via l'adresse IP du nœud et le port attribué à ce service. Pour trouver ce port, exécutez :

```bash
kubectl get sservices wordpress
```

Vous devriez voir une sortie similaire à ceci :

```
NAME        TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress   NodePort   10.96.0.1    <none>        80:30001/TCP   5m
```

Ici, `30001` est le port exposé, donc vous pouvez accéder à WordPress en ouvrant un navigateur et en accédant à `http://<node-ip>:30001`.

### 5. Vérifications finales
- Vérifiez que les pods pour MySQL et WordPress sont en cours d'exécution :
  ```bash
  kubectl get pods
  ```
  
- Vérifiez les services exposés :
  ```bash
  kubectl get service
  ```

- Si vous rencontrez des problèmes, consultez les logs des pods :
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


