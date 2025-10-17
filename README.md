# Movies Watchlist - Application Conteneurisée avec Kubernetes

- Jules Chevallier

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

### Étape 1 : Récupérer l'image Docker

#### Option A : Télécharger depuis Docker Hub (recommandé)

```bash
# Télécharger l'image depuis Docker Hub
docker pull julesche/movies-watchlist:latest

# Taguer l'image localement (optionnel)
docker tag julesche/movies-watchlist:latest movies-watchlist:latest
```

#### Option B : Construire l'image localement

```bash
# Construire l'image Docker de l'application
docker build -t movies-watchlist:latest .
```

### Étape 2 : Charger l'image dans minikube (pour les tests locaux)

```bash
# Si vous utilisez minikube avec l'image depuis Docker Hub
minikube image load julesche/movies-watchlist:latest

# Ou si vous avez construit l'image localement
minikube image load movies-watchlist:latest
```

### Étape 3 : Déployer sur Kubernetes

```bash
# Appliquer tous les manifestes Kubernetes
kubectl apply -f k8s/

# Ou appliquer dans l'ordre recommandé :
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/mongodb-pv.yaml
kubectl apply -f k8s/mongodb-deployment.yaml
kubectl apply -f k8s/app-deployment.yaml
kubectl apply -f k8s/ingress.yaml
```

### Étape 4 : Vérifier le déploiement

```bash
# Vérifier que tous les pods sont en cours d'exécution
kubectl get pods -n movies-watchlist

# Vérifier les services
kubectl get services -n movies-watchlist

# Obtenir l'URL d'accès (avec minikube)
minikube service movies-watchlist-service -n movies-watchlist --url
```

### Étape 5 : Accéder à l'application

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
│   ├── mongodb-pv.yaml        # Volume persistant MongoDB
│   ├── mongodb-deployment.yaml # Déploiement MongoDB
│   ├── app-deployment.yaml    # Déploiement de l'application
│   └── ingress.yaml           # Configuration Ingress
└── README.md                  # Ce fichier
```

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
- **ConfigMaps/Secrets** : Configuration via variables d'environnement
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
