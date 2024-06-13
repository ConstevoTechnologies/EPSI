# Exercice 7 : Gestion des ressources

## Objectif
Cet exercice vise à vous familiariser avec la gestion des ressources dans Kubernetes en configurant les requêtes et les limites CPU et mémoire pour un pod. Vous apprendrez comment allouer efficacement les ressources aux pods pour garantir la performance et la stabilité de vos applications.

## Étapes à suivre

### 1. Configurer les requêtes CPU et mémoire

Les requêtes CPU et mémoire sont utilisées pour garantir qu'un pod dispose d'une quantité minimale de ressources pour fonctionner correctement. Lorsque vous spécifiez des requêtes de ressources, Kubernetes s'assure que chaque nœud dispose des ressources nécessaires pour exécuter les pods.

#### Tâche

- Configurez les requêtes CPU et mémoire pour un pod.

#### Exemple

Créez un fichier nommé `nginx-requests.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-requests
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
```

#### Commande

Appliquez le fichier de configuration :

```bash
kubectl apply -f nginx-requests.yaml
```

### 2. Configurer les limites CPU et mémoire

Les limites CPU et mémoire sont utilisées pour restreindre la quantité maximale de ressources qu'un pod peut utiliser. Lorsque vous spécifiez des limites de ressources, Kubernetes s'assure qu'un pod ne dépasse pas ces limites, ce qui évite qu'un pod utilise toutes les ressources disponibles d'un nœud.

#### Tâche

- Configurez les limites CPU et mémoire pour un pod.

#### Exemple

Créez un fichier nommé `nginx-limits.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-limits
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### Commande

Appliquez le fichier de configuration :

```bash
kubectl apply -f nginx-limits.yaml
```

## Vérification

### Vérifier les requêtes et les limites

Utilisez la commande suivante pour vérifier que les requêtes et les limites ont été correctement appliquées :

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].resources}'
```

Remplacez `<pod-name>` par `nginx-requests` ou `nginx-limits` pour voir les ressources configurées.

### Supprimer les pods

Pour nettoyer les ressources créées après l'exercice, vous pouvez supprimer les pods avec les commandes suivantes :

```bash
kubectl delete pod nginx-requests
kubectl delete pod nginx-limits
```

## Conclusion

En complétant cet exercice, vous aurez appris à configurer les requêtes et les limites de ressources pour des pods dans Kubernetes, garantissant ainsi une meilleure gestion des ressources et une performance optimisée pour vos applications.