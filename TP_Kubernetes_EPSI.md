# Section 1: Applicatifs

## Description des différents objets de l'API de Kubernetes et manipulation avec des exemples concrets

### Namespace
Un **namespace** est une méthode pour diviser les ressources d'un cluster Kubernetes en plusieurs espaces logiques. C'est utile pour organiser et séparer des ensembles de ressources de manière propre.

#### Créer, supprimer, modifier un namespace
- **Créer un namespace:**
```bash
kubectl create namespace mon-namespace
```

- **Supprimer un namespace:**
```bash
kubectl delete namespace mon-namespace
```

- **Modifier un namespace (ajout de labels):**
```bash
kubectl label namespace mon-namespace environnement=dev
```

### Pod
Un **pod** est la plus petite unité de déploiement dans Kubernetes, représentant un ou plusieurs conteneurs qui partagent le même réseau et espace de stockage.

#### Créer un pod nginx exposant un port
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
```bash
kubectl apply -f nginx-pod.yaml
```

#### Accéder au pod via kubectl port-forward
```bash
kubectl port-forward pod/nginx-pod 8080:80
```

#### Supprimer le pod
```bash
kubectl delete pod nginx-pod
```

### ReplicaSet
Un **ReplicaSet** assure qu'un nombre spécifié de répliques d'un pod sont en cours d'exécution à tout moment.

#### Créer un ReplicaSet créant un ou deux pods nginx
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 2
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
```bash
kubectl apply -f nginx-replicaset.yaml
```

#### Supprimer le ReplicaSet
```bash
kubectl delete replicaset nginx-replicaset
```

### Deployment/DaemonSet/StatefulSet

#### Créer un premier StatefulSet simple (ex: MariaDB)
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
spec:
  serviceName: "mariadb"
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root_password"
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mariadb-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
```bash
kubectl apply -f mariadb-statefulset.yaml
```

### Variables d'environnement (envvars)

#### Créer un premier déploiement simple (ex: WordPress) qui utilise le StatefulSet pour stocker ses bases et tables
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: "mariadb"
        - name: WORDPRESS_DB_PASSWORD
          value: "root_password"
```
```bash
kubectl apply -f wordpress-deployment.yaml
```

### Utilité d'un DaemonSet
Un **DaemonSet** assure que tous (ou certains) nœuds exécutent une copie d'un pod. Utilisé pour des tâches comme les logs, monitoring, etc.

#### Exemple: Installation de MetalLB (load-balancer)
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: metallb
spec:
  selector:
    matchLabels:
      app: metallb
  template:
    metadata:
      labels:
        app: metallb
    spec:
      containers:
      - name: metallb
        image: metallb/metallb:v0.9.3
```
```bash
kubectl apply -f metallb-daemonset.yaml
```

### Ressources

#### Requêtes CPU et mémoire
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
```

#### Limites CPU et mémoire
```yaml
resources:
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### Mise à jour des applicatifs

#### Stratégies de mise à jour (RollingUpdate, Recreate)

- **RollingUpdate:**
  Met à jour les pods petit à petit pour garantir une disponibilité continue.
  ```yaml
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  ```

- **Recreate:**
  Supprime tous les pods existants avant de créer les nouveaux.
  ```yaml
  strategy:
    type: Recreate
  ```
