# Volumes
Dans Kubernetes, chaque conteneur possède un **filesystem temporaire** :

* isolé des autres conteneurs
* supprimé après redémarrage

Donc les données sont perdues.

Un **volume** est un répertoire partagé :

* entre plusieurs conteneurs d’un même Pod
* qui peut persister selon le type

<p align="center">
  <img src="volll.png" alt="volll" width="700"/>
</p>

#### Objectifs des volumes

* **Persistance des données** → éviter la perte après restart
* **Partage de données** → communication entre conteneurs (sidecar)

---

#### Types de volumes (important examen)

* **emptyDir** → temporaire (durée de vie du Pod)
* **hostPath** → stockage du node

#### Principe d’utilisation

2 étapes :

1. Déclarer le volume

```yaml
spec.volumes
```

2. Le monter dans le conteneur

```yaml
spec.containers[].volumeMounts
```
Le lien se fait par le **name**

#### Exemple: volume partagé (emptyDir)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  containers:
  - name: nginx
    image: nginx:1.27.1
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: sidecar
    image: busybox:1.37.0
    volumeMounts:
    - name: shared-data
      mountPath: /data
```

```bash
kubectl apply -f pod-with-volume.yaml
```

```bash
kubectl get pod business-app
```

---

#### Test du volume

Entrer dans nginx :

```bash
kubectl exec -it business-app -c nginx -- /bin/sh
```

Créer un fichier :

```bash
cd /usr/share/nginx/html
touch example.html
ls
```

Résultat :

```text
example.html
```

Le volume `emptyDir` est :

* initialisé vide
* partagé entre conteneurs
* supprimé quand le Pod est supprimé

#### Exemple : Volume hostPath

Le volume **hostPath** permet de monter un fichier ou un dossier du **node (machine hôte)** dans un conteneur.

* dépend du node
* non portable
* utilisé surtout pour debug ou cas spécifiques

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: host-volume
      mountPath: /data
  volumes:
  - name: host-volume
    hostPath:
      path: /tmp
      type: Directory
```

* `/tmp` → dossier sur le node
* `/data` → monté dans le conteneur
* `type: Directory` → doit exister sur le node

#### Test

Entrer dans le Pod :
```bash
kubectl exec -it hostpath-pod -- sh
```
Créer un fichier :
```bash
cd /data
touch test.txt
```
Ce fichier existe aussi sur le node :
```bash
ls /tmp
```
# LAB

```bash
Create a Pod YAML manifest with two containers that use the image alpine:3.22.2 . Provide a command for both containers that keeps them running forever.
Define a volume of type emptyDir for the Pod. Container 1 should mount the volume to path /etc/a, and Container 2 should mount the volume to path /etc/b.
Open an interactive shell for Container 1 and create the directory data in the mount path. Navigate to the directory and create the file hello.txt with the contents “Hello World.” Exit out of the container.
Open an interactive shell for Container 2 and navigate to the directory /etc/b/data. Inspect the contents of file hello.txt. Exit out of the container.
```