# Exercice 8 : Utilisation de Kustomize pour Organiser les Fichiers YAML de Kubernetes

## Objectif
Cet exercice vise à vous familiariser avec l'utilisation de Kustomize, un outil intégré à `kubectl` pour personnaliser les fichiers YAML de Kubernetes sans duplication. Vous apprendrez comment organiser et gérer les configurations de manière plus efficace et modulaire.

## Étapes à suivre

### 1. Installer Kustomize

Kustomize est intégré à `kubectl` à partir de la version 1.14. Assurez-vous que vous avez une version compatible :

```bash
kubectl version --client
```

### 2. Préparer la Structure des Répertoires

Créez une structure de répertoires pour votre projet Kubernetes :

```bash
mkdir -p myapp/base
mkdir -p myapp/overlays/dev
mkdir -p myapp/overlays/prod
```

### 3. Créer les Fichiers de Base

Dans le répertoire `myapp/base`, créez les fichiers suivants :

#### Déploiement Nginx (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

#### Service Nginx (`service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

#### Fichier Kustomization (`kustomization.yaml`)

Créez un fichier nommé `kustomization.yaml` dans le répertoire `myapp/base` avec le contenu suivant :

```yaml
resources:
  - deployment.yaml
  - service.yaml
```

### 4. Créer les Overlays pour les Environnements

#### Overlay Dev (`myapp/overlays/dev/kustomization.yaml`)

Créez un fichier nommé `kustomization.yaml` dans le répertoire `myapp/overlays/dev` avec le contenu suivant :

```yaml
resources:
  - ../../base

namePrefix: dev-
```

#### Overlay Prod (`myapp/overlays/prod/kustomization.yaml`)

Créez un fichier nommé `kustomization.yaml` dans le répertoire `myapp/overlays/prod` avec le contenu suivant :

```yaml
resources:
  - ../../base

namePrefix: prod-
```

### 5. Appliquer les Configurations

#### Appliquer la configuration pour l'environnement de développement :

```bash
kubectl apply -k myapp/overlays/dev
```

#### Appliquer la configuration pour l'environnement de production :

```bash
kubectl apply -k myapp/overlays/prod
```

### 6. Vérifier les Déploiements

Vérifiez que les déploiements ont été créés avec les préfixes appropriés :

```bash
kubectl get deployments
kubectl get services
```