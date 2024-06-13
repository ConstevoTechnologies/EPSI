# Installation de MetalLB sur Minikube avec DaemonSet

## Objectif
Installer MetalLB, un load balancer pour les clusters Kubernetes, en utilisant un DaemonSet sur Minikube.

## Prérequis
- Minikube installé et configuré
- `kubectl` configuré pour interagir avec votre cluster Minikube

## Étape 1 : Installer Minikube

Si ce n'est pas déjà fait, installez Minikube en suivant les instructions officielles [ici](https://minikube.sigs.k8s.io/docs/start/).

## Étape 2 : Démarrer Minikube

Démarrez Minikube avec un profile spécifique pour éviter les conflits avec d'autres configurations.

```bash
minikube start --profile=metallb
```

## Étape 3 : Activer l'addon de configuration de MetalLB

Minikube fournit un addon MetalLB pour simplifier l'installation. Vous pouvez l'activer en utilisant la commande suivante :

```bash
minikube addons enable metallb --profile=metallb
```

Cette commande installera MetalLB et le configurera automatiquement.

## Étape 4 : Configurer MetalLB

Une fois que MetalLB est installé, vous devez le configurer pour définir le pool d'adresses IP qu'il utilisera pour allouer aux services de type LoadBalancer.

Créez un fichier de configuration `metallb-config.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.49.240-192.168.49.250
```

Appliquez cette configuration :

```bash
kubectl apply -f metallb-config.yaml
```

Note : Les adresses IP dans `addresses` doivent être dans le même sous-réseau que celui de Minikube. Vous pouvez trouver la plage d'adresses IP en utilisant la commande `minikube ip` et ajuster en conséquence.

## Étape 5 : Vérifier l'installation de MetalLB

Pour vérifier que MetalLB a été correctement installé et configuré, utilisez les commandes suivantes :

```bash
kubectl get pods -n metallb-system
kubectl get services
```

## Utiliser MetalLB avec un Service de type LoadBalancer

Pour tester MetalLB, créez un service de type LoadBalancer. Créez un fichier nommé `nginx-service.yaml` avec le contenu suivant :

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
  type: LoadBalancer
```

Et un fichier `nginx-deployment.yaml` pour déployer un pod Nginx :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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

Appliquez les fichiers :

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

### Conclusion

En suivant ces étapes, vous avez installé et configuré MetalLB sur Minikube en utilisant un DaemonSet. Vous pouvez maintenant créer des services de type LoadBalancer et MetalLB allouera des adresses IP externes pour ces services.