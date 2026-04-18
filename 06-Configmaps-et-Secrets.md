# ConfigMaps et Secrets
Kubernetes propose deux objets pour gérer la configuration : ConfigMap et Secret.
Ils sont indépendants des Pods, ce qui permet de modifier la configuration sans redéployer les applications.

Ils stockent des données sous forme de paires clé-valeur, qui peuvent être utilisées dans les conteneurs soit comme variables d’environnement, soit comme fichiers via des volumes.

<p align="center">
  <img src="configmap.png" alt="configmap" width="700"/>
</p>

* **ConfigMap** → stocke des données en texte clair (config, URLs, JSON…)
* **Secret** → stocke des données sensibles encodées en Base64 (passwords, API keys…)

Les applications utilisent des données de configuration (URL, paramètres réseau…) qui varient selon l’environnement et peuvent être partagées entre plusieurs Pods.  

Au lieu de les dupliquer, Kubernetes permet de les centraliser dans un ConfigMap, utilisable par plusieurs Pods, ce qui facilite leur gestion et leur modification en un seul endroit.  

## Création d’un ConfigMap

Vous pouvez créer un ConfigMap en utilisant la commande impérative `create configmap`. Cette commande nécessite de fournir la source des données en option. Kubernetes distingue quatre options différentes, comme montré ci-dessous.

### Options de source pour un ConfigMap

| Option            | Exemple                       | Description                                                 |
| ----------------- | ----------------------------- | ----------------------------------------------------------- |
| `--from-literal`  | `--from-literal=locale=en_US` | Valeurs littérales (paires clé-valeur en texte clair)       |
| `--from-env-file` | `--from-env-file=config.env`  | Fichier contenant des variables d’environnement (KEY=value) |
| `--from-file`     | `--from-file=appconfig.json`  | Fichier avec contenu (json, yaml,xml..)                     |
| `--from-file`     | `--from-file=config-dir`      | Répertoire contenant un ou plusieurs fichiers               |

#### Exemple avec --from-literal

```bash
kubectl create configmap app-config \
--from-literal=APP_ENV=prod \
--from-literal=APP_DEBUG=false
```

---

#### Exemple avec --from-env-file

Fichier `config.env` :

```bash
APP_ENV=prod
APP_DEBUG=false
```

Commande :

```bash
kubectl create configmap app-config --from-env-file=config.env
```

---

#### Exemple avec --from-file (fichier)

Fichier `appconfig.json` :

```json
{
  "env": "prod",
  "debug": false
}
```

Commande :

```bash
kubectl create configmap app-config --from-file=appconfig.json
```

---

#### Exemple avec --from-file (répertoire)

```bash
kubectl create configmap app-config --from-file=config-dir/
```

---

Il est facile de confondre `--from-env-file` et `--from-file`.
L’option `--from-env-file` attend un fichier contenant des variables d’environnement au format `KEY=value`, séparées par des retours à la ligne.

L’option `--from-env-file` n’impose pas ces conventions.
L’option `--from-file`, quant à elle, permet de lire un fichier ou un répertoire contenant du contenu arbitraire (JSON, YAML, XML, etc.).

---

#### Exemple de création

```bash
kubectl create configmap db-config \
--from-literal=DB_HOST=mysql-service \
--from-literal=DB_USER=backend
```

Résultat :

```
configmap/db-config created
```

---

#### YAML généré

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  DB_HOST: mysql-service
  DB_USER: backend
```

Le ConfigMap définit les paires clé-valeur dans la section `data`.
Un ConfigMap ne possède pas de section `spec`.

Vous pouvez remarquer que les clés suivent les conventions des variables d’environnement, car elles sont souvent utilisées comme telles dans les conteneurs.
---

## Utiliser un ConfigMap comme variables d’environnement

Une fois le ConfigMap créé, on peut injecter ses paires clé-valeur dans un conteneur sous forme de variables d’environnement en utilisant `envFrom` avec `configMapRef`.

---

#### Exemple

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    envFrom:
    - configMapRef:
        name: db-config
```

#### Vérification

```bash
kubectl exec backend -- env
```

Exemple de résultat :

```text
DB_HOST=mysql-service
DB_USER=backend
```
Les données du ConfigMap apparaissent comme variables d’environnement dans le conteneur.

---

# Monter un ConfigMap comme volume

On peut aussi utiliser un ConfigMap comme **volume** pour exposer ses données sous forme de fichiers dans le conteneur.

- Le ConfigMap est monté comme des fichiers dans le conteneur
- Kubernetes peut mettre à jour les fichiers automatiquement

* Les variables d’environnement issues d’un ConfigMap sont injectées au démarrage du conteneur et ne changent plus sans redémarrage.

* Un ConfigMap monté en volume, lui, permet de mettre à jour les données dynamiquement, sans redémarrer le Pod.
```bash
kubectl edit configmap mon-config
```

#### Exemple

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: db-config
```

---

## Vérification

```bash
kubectl exec backend -- ls /etc/config
```

```bash
kubectl exec backend -- cat /etc/config/DB_HOST
```

Chaque clé du ConfigMap devient un fichier dans `/etc/config`

---
# Secret
## Créer un Secret

On peut créer un **Secret** avec la commande impérative `kubectl create secret`.
Il faut préciser un **type de Secret** avec un sous-commande.



### Types de Secrets

| Option CLI      | Description                                                 | Type interne            |
| --------------- | ----------------------------------------------------------- | ----------------------- |
| generic         | Crée un Secret à partir de valeurs, fichiers ou répertoires | Opaque                  |
| docker-registry | Secret pour accéder à un registre Docker privé              | kubernetes.io/dockercfg |
| tls             | Secret pour certificats TLS                                 | kubernetes.io/tls       |

Le type le plus utilisé est **generic**.



#### Sources des données

| Option          | Exemple                            | Description       |
| --------------- | ---------------------------------- | ----------------- |
| --from-literal  | `--from-literal=password=secret`   | paires clé-valeur |
| --from-env-file | `--from-env-file=config.env`       | fichier env       |
| --from-file     | `--from-file=id_rsa=~/.ssh/id_rsa` | fichier           |
| --from-file     | `--from-file=config-dir`           | dossier           |



#### créer un Secret

Créer un Secret de type generic avec des valeurs simples.

```bash
kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=secret
```

---

#### Vérification

```bash
kubectl get secrets
```

```bash
kubectl describe secret db-secret
```
 Le Secret stocke les données en Base64.

```bash
kubectl describe secret db-secret
```
## Créer un Secret via Yaml

Si tu crées un Secret avec un fichier YAML, les valeurs doivent être encodées en Base64.


#### Encoder une valeur

```bash id="s0j3pz"
echo -n 's3cre!' | base64
```

Résultat :

```text
czNjcmUh
```

---

#### Exemple de Secret en YAML

```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: czNjcmUh
```

`password` contient la valeur encodée en Base64.

---

#### Vérification

```bash
kubectl get secret db-secret -o yaml
```

Pour décoder :

```bash
echo 'czNjcmUh' | base64 --decode
```

Résultat :

```text
s3cre!
```
## Utiliser un Secret comme variables d’environnement

L’utilisation d’un Secret comme variables d’environnement fonctionne de la même manière qu’un ConfigMap, mais en utilisant `secretRef`.

Kubernetes décode automatiquement les valeurs Base64, donc l’application n’a pas besoin de le faire.

---

#### Exemple

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    envFrom:
    - secretRef:
        name: secret-basic-auth
```

#### Vérification

```bash 
kubectl exec backend -- env
```

Les valeurs du Secret apparaissent directement (déjà décodées) comme variables d’environnement dans le conteneur.



#### Monter un Secret comme volume

On peut monter un Secret comme un volume pour exposer ses données sous forme de fichiers dans le conteneur.


#### Créer le Secret (clé SSH)

```bash
cp ~/.ssh/id_rsa ssh-privatekey
```

```bash 
kubectl create secret generic secret-ssh-auth --from-file=ssh-privatekey --type=kubernetes.io/ssh-auth
```


#### Définir le Pod

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    volumeMounts:
    - name: ssh-volume
      mountPath: /var/app
      readOnly: true
  volumes:
  - name: ssh-volume
    secret:
      secretName: secret-ssh-auth
```

---

#### Vérification

```bash
kubectl exec backend -- ls /var/app
```
Le fichier `ssh-privatekey` sera disponible dans le conteneur.

#### Vérification détaillée

```bash
kubectl exec -it backend -- /bin/sh
```

```bash 
ls -1 /var/app
```

```bash 
cat /var/app/ssh-privatekey
```

Résultat :

```text
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,8734C9153079F2E8497C8075289EBBF1
...
-----END RSA PRIVATE KEY-----
```

Le contenu du fichier **n’est pas encodé en Base64** dans le conteneur.

---

#### Remarques importantes

* Les fichiers montés depuis un Secret sont **en lecture seule**
* Pour un Secret → on utilise `secretName`
* (contrairement à ConfigMap → `name`)


# LAB
```bash 
1. In this exercise, you will first create a ConfigMap from a YAML configuration file as a source. Later, you’ll create a Pod, consume the
ConfigMap as volume, and inspect the key-value pairs as files.
Navigate to the directory app-a/ch10/configmap of the checked-out
GitHub repository bmuschko/cka-study-guide. Inspect the YAML configuration file named application.yaml.
Create a new ConfigMap named app-config from that file.
Create a Pod named backend that consumes the ConfigMap as volume
at the mount path /etc/config. The container runs the image
nginx:1.23.4-alpine .
Shell into the Pod and inspect the file at the mounted volume path.
2. You will first create a Secret from literal values in this exercise. Next,
you’ll create a Pod and consume the Secret as environment variables.
Finally, you’ll print out its values from within the container.
Create a new Secret named db-credentials with the key-value pair
db-password=passwd .
Create a Pod named backend that uses the Secret as an environment
variable named DB_PASSWORD and runs the container with the image
nginx:1.23.4-alpine .
Shell into the Pod and print out the created environment variables.
You should be able to find the DB_PASSWORD variable.
```