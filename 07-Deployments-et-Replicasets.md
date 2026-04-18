 # Deployment
<p align="center">
  <img src="dep2.png" alt="dep2" width="600"/>
</p>

apiVersion: apps/v1
kind: Deployment
metadata:
 name: app-cache
 labels:
 app: app-cache
spec:
 replicas: 4
 selector:
 matchLabels:
 app: app-cache
 template:
 metadata:
 labels:
 app: app-cache
 spec:
 containers:
 - name: memcached
 image: memcached:1.6.8

### Labels et sélection des Pods

Lors de la création d’un Deployment, le label `app` est utilisé par défaut et apparaît dans :

* `metadata.labels` → label du Deployment (non utilisé pour sélectionner les Pods)
* `spec.selector.matchLabels` → utilisé pour sélectionner les Pods
* `spec.template.metadata.labels` → labels appliqués aux Pods
<p align="center">
  <img src="dep3.png" alt="kube" width="700"/>
</p>

**Important** :
Pour que la sélection fonctionne, les valeurs de
`spec.selector.matchLabels` et `spec.template.metadata.labels` doivent être identiques.
Sinon → erreur lors de la création.

Le champ `metadata.labels` peut être différent, il n’est pas utilisé pour le mapping Deployment → Pods.

---

#### Rolling Update (mise à jour progressive)

Un Deployment gère les mises à jour via des **ReplicaSets**.
<p align="center">
  <img src="dep4.png" alt="kube" width="600"/>
</p>

**Principe** :

* ancien ReplicaSet = ancienne version
* nouveau ReplicaSet = nouvelle version
* Kubernetes :

  * crée le nouveau ReplicaSet
  * démarre de nouveaux Pods
  * réduit progressivement les anciens Pods

Pendant la mise à jour :

* les deux versions coexistent
* le Service envoie le trafic vers les deux

---

#### Mise à jour de l’image

```bash
kubectl set image deployment app-cache memcached=memcached:1.6.10
```

Cette commande modifie uniquement l’image du conteneur dans le Pod template.

---

#### Vérifier le rollout

```bash
kubectl rollout status deployment app-cache
```
Affiche la progression :

* combien de Pods sont mis à jour
* quand le déploiement est terminé

---

#### Historique des versions (revisions)

```bash
kubectl rollout history deployment app-cache
```

**Chaque modification crée une revision**

* revision 1 → version initiale
* revision 2 → nouvelle image

Voir détail :

```bash 
kubectl rollout history deployment app-cache --revision=2
```

Permet de voir :

* image utilisée
* configuration du Pod

---

#### Rolling Update comportement

 **Important** :

* Kubernetes augmente progressivement les nouveaux Pods
* diminue progressivement les anciens Pods
* garantit disponibilité continue

---

## Changer la stratégie

```yaml
spec:
  strategy:
    type: Recreate
```

**RollingUpdate (default)** → zéro downtime
**Recreate** → supprime tout puis recrée → downtime

---

## Rollback (retour arrière)

```bash 
kubectl rollout undo deployment app-cache
```

Ou version spécifique :

```bash 
kubectl rollout undo deployment app-cache --to-revision=1
```
* Kubernetes revient à l’ancienne version
* crée un nouveau ReplicaSet basé sur l’ancienne config

```bash 
kubectl rollout history deployment app-cache
```

* ne garde pas l’ancienne revision telle quelle
* recrée une nouvelle revision avec l’ancien contenu

