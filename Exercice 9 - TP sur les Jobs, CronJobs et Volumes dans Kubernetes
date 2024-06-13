## Objectif
Cet exercice vise à vous familiariser avec les concepts de Jobs et CronJobs dans Kubernetes et à vous apprendre à créer et utiliser des volumes pour stocker des données persistantes.

## Étapes à suivre

### 1. Créer un Volume Persistant (PV) et un Persistent Volume Claim (PVC)

Les volumes persistants permettent de stocker des données de manière persistante, même si les pods qui les utilisent sont supprimés ou redémarrés.

#### Créer un Volume Persistant (PV)

Créez un fichier nommé `pv.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

#### Créer un Persistent Volume Claim (PVC)

Créez un fichier nommé `pvc.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

#### Appliquer les configurations

Appliquez les configurations pour créer le PV et le PVC :

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

### 2. Créer un Job

Les Jobs dans Kubernetes permettent d'exécuter une ou plusieurs tâches jusqu'à ce qu'elles soient complétées avec succès.

#### Créer un fichier de configuration pour le Job

Créez un fichier nommé `example-job.yaml` avec le contenu suivant :

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
      - name: example
        image: busybox
        command: ["sh", "-c", "echo 'Hello, Kubernetes!' && sleep 30"]
        volumeMounts:
        - mountPath: /mnt/data
          name: example-storage
      restartPolicy: OnFailure
      volumes:
      - name: example-storage
        persistentVolumeClaim:
          claimName: example-pvc
```

#### Appliquer la configuration

Appliquez la configuration pour créer le Job :

```bash
kubectl apply -f example-job.yaml
```

#### Vérifier le Job

Vérifiez que le Job a été créé et qu'il s'exécute :

```bash
kubectl get jobs
kubectl get pods
kubectl logs <job-pod-name>
```

### 3. Créer un CronJob

Les CronJobs dans Kubernetes permettent de planifier l'exécution répétée de tâches à des intervalles spécifiés, similaires aux tâches cron en Unix/Linux.

#### Créer un fichier de configuration pour le CronJob

Créez un fichier nommé `example-cronjob.yaml` avec le contenu suivant :

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/1 * * * *" # S'exécute toutes les minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: example
            image: busybox
            command: ["sh", "-c", "echo 'Hello, Kubernetes CronJob!' > /mnt/data/cronjob.txt"]
            volumeMounts:
            - mountPath: /mnt/data
              name: example-storage
          restartPolicy: OnFailure
          volumes:
          - name: example-storage
            persistentVolumeClaim:
              claimName: example-pvc
```

#### Appliquer la configuration

Appliquez la configuration pour créer le CronJob :

```bash
kubectl apply -f example-cronjob.yaml
```

#### Vérifier le CronJob

Vérifiez que le CronJob a été créé et qu'il planifie des jobs :

```bash
kubectl get cronjobs
kubectl get jobs --watch
kubectl logs <cronjob-pod-name>
```
