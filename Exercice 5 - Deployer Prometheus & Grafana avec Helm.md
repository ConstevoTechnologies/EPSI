# Supervision du Cluster Kubernetes avec kube-prometheus-stack et Helm

Dans cet exercice, vous allez apprendre à déployer et configurer kube-prometheus-stack (qui inclut Prometheus et Grafana) pour superviser un cluster Kubernetes. Nous utiliserons Helm pour le déploiement et exposerons Prometheus et Grafana à l'extérieur via ingress-nginx.

## Prérequis

- Un cluster Kubernetes en fonctionnement
- Helm installé et configuré
- Ingress-nginx installé et configuré
- Minikube ou une autre distribution Kubernetes pour les tests locaux

## Étapes à suivre

### 1. Installer Helm

Si Helm n'est pas déjà installé, suivez les instructions officielles pour l'installer : [Install Helm](https://helm.sh/docs/intro/install/)

### 2. Ajouter les Charts Repositories

Ajoutez le repository Helm pour kube-prometheus-stack :

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

### 3. Créer un Namespace pour Prometheus

Créez un namespace dédié pour kube-prometheus-stack :

```bash
kubectl create namespace monitoring
```

### 4. Déployer kube-prometheus-stack

Utilisez Helm pour installer kube-prometheus-stack dans le namespace `monitoring` :

```bash
helm install prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring
```

### 5. Vérifier les Déploiements

Vérifiez que tous les pods sont en cours d'exécution dans le namespace `monitoring` :

```bash
kubectl get pods -n monitoring
```

Vous devriez voir des pods pour Prometheus, Grafana, Alertmanager, et d'autres composants de la stack de monitoring.

### 6. Configurer Ingress pour Prometheus et Grafana

Nous allons exposer Prometheus et Grafana en utilisant ingress-nginx.

#### Créer un Ingress pour Prometheus

Créez un fichier nommé `prometheus-ingress.yaml` avec le contenu suivant :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: prometheus.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-stack-kube-prometheus-prometheus
            port:
              number: 9090
```

#### Créer un Ingress pour Grafana

Créez un fichier nommé `grafana-ingress.yaml` avec le contenu suivant :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: grafana.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-stack-grafana
            port:
              number: 80
```

#### Appliquer les Ingress

Appliquez les fichiers de configuration pour créer les Ingress :

```bash
kubectl apply -f prometheus-ingress.yaml
kubectl apply -f grafana-ingress.yaml
```

### 7. Mettre à jour le fichier hosts

Ajoutez des entrées dans votre fichier `/etc/hosts` pour mapper les noms de domaine locaux à l'adresse IP de votre cluster Minikube ou à l'adresse IP de l'Ingress Controller.

```plaintext
<MINIKUBE_IP> prometheus.local
<MINIKUBE_IP> grafana.local
```

Pour obtenir l'adresse IP de Minikube :

```bash
minikube ip
```

### 8. Accéder à Prometheus et Grafana

Ouvrez un navigateur web et accédez à :

- Prometheus : `http://prometheus.local`
- Grafana : `http://grafana.local`

### 9. Configurer Grafana

Pour Grafana, connectez-vous avec les identifiants par défaut :

- Username: `admin`
- Password: `prom-operator`

Vous pouvez ensuite ajouter des dashboards pour superviser votre cluster Kubernetes.
