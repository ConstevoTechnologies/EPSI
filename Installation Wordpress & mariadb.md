Deploy WordPress and MariaDB

Ensure you have the following YAML files for deploying WordPress and MariaDB.

#### `mariadb-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mariadb-config
data:
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wordpress
  MYSQL_PASSWORD: wordpress
  MYSQL_ROOT_PASSWORD: root_password
```

#### `mariadb-statefulset.yaml`

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
        image: mariadb:10.5
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_PASSWORD
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_ROOT_PASSWORD
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

#### `mariadb-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mariadb
```

#### `wordpress-deployment.yaml`

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
        image: wordpress:latest
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: "mariadb:3306"
        - name: WORDPRESS_DB_NAME
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_DATABASE
        - name: WORDPRESS_DB_USER
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_USER
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: mariadb-config
              key: MYSQL_PASSWORD
```

#### `wordpress-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
spec:
  selector:
    app: wordpress
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### `wordpress-ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wordpress-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: wordpress.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wordpress
            port:
              number: 80
```

### Step 5: Apply the Configurations

Apply the YAML files to deploy the application:

```bash
kubectl apply -f mariadb-configmap.yaml
kubectl apply -f mariadb-statefulset.yaml
kubectl apply -f mariadb-service.yaml
kubectl apply -f wordpress-deployment.yaml
kubectl apply -f wordpress-service.yaml
kubectl apply -f wordpress-ingress.yaml
```

### Step 6: Verify the Service

Check the services to see if MetalLB has assigned an IP address to the LoadBalancer service:

```bash
kubectl get services
```

You should see an external IP assigned to the `wordpress` service.
