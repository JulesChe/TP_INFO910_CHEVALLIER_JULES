# Movies Watchlist - Application Conteneurisée avec Kubernetes

- Jules Chevallier

**Image Docker disponible sur GitHub Container Registry**

```bash
docker pull ghcr.io/julesche/movies-watchlist:latest
```

## Description de l'application

Cette application est un gestionnaire de films à regarder (watchlist) développé avec Node.js et MongoDB. Elle permet aux utilisateurs de :

- Ajouter de nouveaux films avec titre, année et genre
- Marquer les films comme vus ou à regarder
- Supprimer des films de la liste
- Visualiser la liste de tous les films avec leur statut
- Voir les statistiques (total de films, films vus, films à regarder)

L'application est composée de deux conteneurs principaux :

1. **Backend Node.js** : API REST qui gère les opérations CRUD sur les films
2. **Base de données MongoDB** : Stockage persistant des données

## Architecture

```
┌─────────────────────┐    ┌─────────────────────┐
│   Frontend Web      │    │   Backend Node.js   │
│   (HTML/CSS/JS)     │◄──►│   (Express + API)   │
└─────────────────────┘    └─────────────────────┘
                                       │
                                       ▼
                           ┌─────────────────────┐
                           │    MongoDB          │
                           │  (Base de données)  │
                           └─────────────────────┘
```

## Comment utiliser l'application

### Interface Web

1. Accédez à l'application via votre navigateur
2. Utilisez le formulaire en haut pour ajouter un nouveau film (titre, année, genre)
3. Cliquez sur "Mark as Watched" pour marquer un film comme vu
4. Cliquez sur "Delete" pour supprimer un film
5. Consultez les statistiques en haut de la page

### API REST

L'application expose également une API REST :

- `GET /api/movies` - Récupérer tous les films
- `POST /api/movies` - Créer un nouveau film
- `PUT /api/movies/:id` - Mettre à jour un film
- `DELETE /api/movies/:id` - Supprimer un film
- `GET /health` - Vérifier l'état de l'application

## Déploiement sur Kubernetes

### Prérequis

- Docker
- Kubernetes (minikube pour les tests locaux)
- kubectl

### Méthode 1 : Déploiement avec l'image pré-buildée (Recommandé)

L'image Docker est disponible sur **GitHub Container Registry** et sera téléchargée automatiquement lors du déploiement.

**Avantage :** Pas besoin de builder l'image localement, le déploiement est immédiat.

```bash
# L'image ghcr.io/julesche/movies-watchlist:latest sera téléchargée automatiquement
# Passez directement à l'étape de déploiement ci-dessous
```

### Méthode 2 : Build local (Pour développement)

Si vous souhaitez modifier et tester l'application :

#### Étape 1 : Construire l'image localement

```bash
# Construire l'image Docker de l'application
docker build -t ghcr.io/julesche/movies-watchlist:latest .
```

#### Étape 2 : Charger dans Minikube (si vous utilisez Minikube)

```bash
# Configurer le daemon Docker de Minikube
eval $(minikube docker-env)

# Construire directement dans Minikube
docker build -t ghcr.io/julesche/movies-watchlist:latest .
```

### Déploiement sur Kubernetes

```bash
# Appliquer tous les manifestes Kubernetes
kubectl apply -f k8s/

# Ou appliquer dans l'ordre recommandé :
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/mongodb-secret.yaml    # Créer le secret en premier
kubectl apply -f k8s/mongodb-pv.yaml
kubectl apply -f k8s/mongodb-deployment.yaml
kubectl apply -f k8s/app-deployment.yaml
kubectl apply -f k8s/ingress.yaml
```

### Vérification du déploiement

```bash
# Vérifier que tous les pods sont en cours d'exécution
kubectl get pods -n movies-watchlist

# Vérifier les services
kubectl get services -n movies-watchlist

# Obtenir l'URL d'accès (avec minikube)
minikube service movies-watchlist-service -n movies-watchlist --url
```

### Accès à l'application

#### Option 1 : Via LoadBalancer (minikube)

```bash
minikube service movies-watchlist-service -n movies-watchlist
```

#### Option 2 : Via port-forward

```bash
kubectl port-forward service/movies-watchlist-service 8080:80 -n movies-watchlist
```

Puis accédez à `http://localhost:8080`

#### Option 3 : Via Ingress (si configuré)

Ajoutez à votre fichier `/etc/hosts` :

```
<MINIKUBE_IP> movies-watchlist.local
```

Puis accédez à `http://movies-watchlist.local`

## Structure des fichiers

```
├── app.js                      # Application Node.js principale
├── package.json                # Dépendances Node.js
├── Dockerfile                  # Image Docker de l'application
├── docker-compose.yml          # Composition Docker pour développement
├── .dockerignore              # Fichiers à ignorer lors du build Docker
├── public/
│   └── index.html             # Interface utilisateur web
├── k8s/                       # Manifestes Kubernetes
│   ├── namespace.yaml         # Namespace dédié
│   ├── mongodb-secret.yaml    # Secrets pour MongoDB (credentials)
│   ├── mongodb-pv.yaml        # Volume persistant MongoDB
│   ├── mongodb-deployment.yaml # Déploiement MongoDB
│   ├── app-deployment.yaml    # Déploiement de l'application
│   └── ingress.yaml           # Configuration Ingress
└── README.md                  # Ce fichier
```

## Image Docker sur GitHub Container Registry

L'image de l'application est publiée sur **GitHub Container Registry (GHCR)**, ce qui facilite le déploiement :

- **URL de l'image** : `ghcr.io/julesche/movies-watchlist:latest`
- **Accès** : Public (aucune authentification requise)
- **Versions disponibles** :
  - `latest` : Version la plus récente
  - `v1.0.0` : Version stable taggée

### Avantages de GHCR

- Téléchargement automatique lors du déploiement Kubernetes
- Pas besoin de builder l'image localement
- Intégré directement avec le dépôt GitHub
- Image toujours à jour avec le code source

### Tester l'image localement

```bash
# Télécharger et lancer l'image
docker run -p 3000:3000 -e MONGODB_URI=mongodb://mongodb:27017/movieswatchlist ghcr.io/julesche/movies-watchlist:latest
```

## Gestion des Secrets Kubernetes

L'application utilise un Secret Kubernetes pour stocker de manière sécurisée les credentials MongoDB. Le fichier `k8s/mongodb-secret.yaml` contient les informations suivantes encodées en base64 :

- Username root MongoDB
- Password root MongoDB
- Nom de la base de données
- Username et password applicatifs (pour usage futur)

### Personnaliser les credentials

Pour modifier les credentials par défaut :

```bash
# Encoder vos propres valeurs en base64
echo -n "votre_nouveau_password" | base64

# Puis modifier le fichier k8s/mongodb-secret.yaml avec les nouvelles valeurs
```

**Note de sécurité** : Dans un environnement de production, les secrets ne devraient jamais être commités dans Git. Utilisez plutôt des solutions comme :
- `kubectl create secret` pour créer des secrets directement
- Sealed Secrets pour chiffrer les secrets
- External Secrets Operator
- HashiCorp Vault

## Développement local

### Avec Docker Compose

```bash
# Lancer l'application avec Docker Compose
docker-compose up -d

# Arrêter l'application
docker-compose down
```

### Sans Docker

```bash
# Installer les dépendances
npm install

# Démarrer MongoDB localement (requis)
# Puis démarrer l'application
npm start
```

## Surveillance et maintenance

### Logs des applications

```bash
# Logs de l'application
kubectl logs -f deployment/movies-watchlist-app -n movies-watchlist

# Logs de MongoDB
kubectl logs -f deployment/mongodb -n movies-watchlist
```

### Mise à l'échelle

```bash
# Augmenter le nombre de répliques de l'application
kubectl scale deployment movies-watchlist-app --replicas=3 -n movies-watchlist
```

### Nettoyage

```bash
# Supprimer tous les objets du namespace
kubectl delete namespace movies-watchlist
```

## Fonctionnalités Kubernetes implémentées

- **Namespace** : Isolation des ressources
- **Deployments** : Gestion des répliques et mises à jour
- **Services** : Exposition des applications
- **PersistentVolumes** : Stockage persistant pour MongoDB
- **Secrets** : Gestion sécurisée des credentials MongoDB (encodés en base64)
- **Probes** : Vérifications de santé (liveness/readiness)
- **Resource Limits** : Limitation des ressources CPU/mémoire
- **Ingress** : Routage HTTP externe
- **LoadBalancer** : Équilibrage de charge

## Fonctionnalités de l'application

- **Interface moderne** : Design attrayant avec dégradés de couleurs
- **Statistiques en temps réel** : Compteur de films totaux, vus et à regarder
- **Formulaire intuitif** : Ajout facile de films avec titre, année et genre
- **Actions rapides** : Boutons pour marquer comme vu ou supprimer
- **Responsive** : S'adapte aux différentes tailles d'écran

## Troubleshooting

### Problèmes courants

1. **Pods en état Pending** : Vérifiez les ressources disponibles
2. **Connexion MongoDB échoue** : Vérifiez que le service MongoDB est actif
3. **Image non trouvée** : Assurez-vous que l'image est chargée dans minikube

### Commandes utiles

```bash
# Décrire un pod pour diagnostiquer
kubectl describe pod <pod-name> -n movies-watchlist

# Accéder au shell d'un conteneur
kubectl exec -it <pod-name> -n movies-watchlist -- /bin/sh

# Vérifier les événements
kubectl get events -n movies-watchlist --sort-by='.lastTimestamp'
```
