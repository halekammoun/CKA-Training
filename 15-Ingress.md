# Ingress

Ce chapitre a exploré l’objectif et la création de la primitive **Service**. Lorsqu’il devient nécessaire d’exposer une application à des consommateurs externes, le choix du type de Service approprié devient crucial. Le choix le plus pratique consiste souvent à créer un Service de type **LoadBalancer**. Un tel Service offre des capacités de load balancing en attribuant une adresse IP externe accessible aux consommateurs en dehors du cluster Kubernetes.

Cependant, utiliser un Service de type **LoadBalancer** pour chaque application accessible de l’extérieur présente des inconvénients. Dans un environnement cloud, chaque Service entraîne la création d’un load balancer externe, ce qui augmente les coûts. De plus, gérer plusieurs objets Service peut devenir complexe, car un nouvel objet doit être créé pour chaque microservice exposé.
<p align="center">
  <img src="image.png" alt="image" width="700"/>
</p>

Pour résoudre ces problèmes, la primitive **Ingress** intervient en fournissant un point d’entrée unique et équilibré pour une application. Un Ingress est capable de router les requêtes HTTP(S) externes vers un ou plusieurs Services dans le cluster, en se basant sur un nom d’hôte DNS et un chemin d’URL.

**Ingress Controller**
- moteur qui applique les règles Ingress
**Ingress**
- règles de routage (host / path)

#### Travailler avec les Ingress

Un Ingress expose des routes HTTP (et éventuellement HTTPS) vers des clients externes via une URL accessible. Les règles définies déterminent comment le trafic est routé.

Dans les environnements cloud, un load balancer externe est souvent utilisé. L’Ingress reçoit une IP publique et peut router vers plusieurs Services selon les chemins d’URL.

Exemple :
- `/app` → App Service
- `/metrics` → Metrics Service



#### Installation de MetalLB (Load Balancer de test)

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```
installe MetalLB dans le cluster

#### Configurer un pool d’IP
```bash
apiVersion: metallb.io/v1beta1  
kind: IPAddressPool  
metadata:  
  name: default-pool  
  namespace: metallb-system  
spec:  
  addresses:  
  - 192.168.1.240-192.168.1.250  
```
plage IP utilisée pour les services LoadBalancer

#### Activer Layer2
```bash
apiVersion: metallb.io/v1beta1  
kind: L2Advertisement  
metadata:  
  name: l2  
  namespace: metallb-system  
```
permet l’annonce des IP sur le réseau

#### Installer un Ingress Controller

Un Ingress nécessite un **controller** pour fonctionner.
Ce controller applique les règles définies.

exemple avec NGINX
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
installe :
- pods controller
- service LoadBalancer
- RBAC

```bash
kubectl get pods -n ingress-nginx  
```
doit être Running  
```bash
kubectl get svc -n ingress-nginx  
```
doit afficher :  
LoadBalancer → EXTERNAL-IP (MetalLB)

#### IngressClass

Lister :
```bash
kubectl get ingressclasses  
```
exemple: nginx  

`ingressClassName` indique : "quel controller doit gérer cet Ingress"

#### Créer une application test

kubectl create deployment web --image=nginx  
kubectl expose deployment web --port=80  

crée pod + service

#### Créer un Ingress
```bash
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: web  
spec:  
  ingressClassName: nginx  
  rules:  
  - host: web.local  
    http:  
      paths:  
      - path: /  
        pathType: Prefix  
        backend:  
          service:  
            name: web  
            port:  
              number: 80  
```
règle :
- host = web.local
- route vers service web

#### Configurer DNS local
```bash
sudo nano /etc/hosts  
```
```bash
192.168.1.241 web.local  
```
IP = celle du LoadBalancer

#### Tester

curl http://web.local  

doit afficher nginx

#### Test sans host (debug)

curl -H "Host: web.local" http://192.168.1.241  

permet de bypass DNS


#### Règles Ingress

| Type | Exemple | Description |
|------|--------|------------|
| host | next.example.com | Si défini, la règle s’applique à ce host |
| paths | /app | Le trafic doit matcher host + path |
| backend | app-service:8080 | Service cible |

#### Types de path

| Type | Description |
|------|------------|
| Exact | correspond exactement |
| Prefix | correspond au début |

Différence : gestion du `/` final


#### Résolution d’erreur

Si un service n’existe pas :

error: 503 service temporarely unavaiable

Solution : créer le Pod + Service

Si 404 error

error: le ingress ne pointe pas a un controller

Solution: ajouter ingressClassName dans l'ingress


#### Ingress avec HTTPS

Générer un certificat auto-signé
```bash
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout key.pem \
  -out cert.pem \
  -subj "/CN=web.local"
```
crée :
- cert.pem
- key.pem

Créer un Secret TLS
```bash
kubectl create secret tls tls-secret \
  --cert=cert.pem \
  --key=key.pem  
```
stocke le certificat dans Kubernetes.  

Créer un Ingress sécurisé
```bash
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: web  
  annotations:  
    nginx.ingress.kubernetes.io/ssl-redirect: "true"  
spec:  
  ingressClassName: nginx  
  tls:  
  - hosts:  
    - web.local  
    secretName: tls-secret  
  rules:  
  - host: web.local  
    http:  
      paths:  
      - path: /  
        pathType: Prefix  
        backend:  
          service:  
            name: web  
            port:  
              number: 80  
```
fonctionnalités :
HTTPS activé
HTTP redirigé vers HTTPS

Tester

#### HTTP

curl http://web.local  

Résultat :

308 Permanent Redirect  

explication :
HTTP → redirigé vers HTTPS

#### HTTPS

curl https://web.local  

Résultat :

SSL certificate problem  

explication :
certificat auto-signé → non trusted

---

#### HTTPS (test forcé)

curl -k https://web.local  

Résultat :
page nginx

Exemple : plusieurs services avec paths

On veut router :
- /app → service app  
- /metrics → service metrics  

#### Ingress avec plusieurs paths

Les paths doivent etre supporté par l'application ou précisement l'image
Déployer services  
```bash
kubectl create deployment app --image=nginx  
kubectl expose deployment app --port=80  
kubectl create deployment metrics --image=nginx  
kubectl expose deployment metrics --port=80  
```
```bash
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: multi-app  
spec:  
  ingressClassName: nginx  
  rules:  
  - host: web.local  
    http:  
      paths:  
      - path: /  
        pathType: Prefix  
        backend:  
          service:  
            name: app  
            port:  
              number: 80  
      - path: /indes.html  
        pathType: Prefix  
        backend:  
          service:  
            name: metrics  
            port:  
              number: 80  
```

Tester
```bash
curl http://web.local/  
curl http://web.local/index.html
```
chaque path est routé vers un service différent  

---

# QUESTION 12 
1. Expose the existing deployment with a service called echo-service using Service Port 8080 type=NodePort
2. Create a new ingress resource named echo in the echo-sound namespace for http://example.org/echo
3. The availability of the Service echo-service can be checked using the following command
curl NODEIP:NODEPORT/echo  
script for lab setup 
```bash
#!/bin/bash
set -e

echo "Creating namespace: echo-sound"
kubectl create ns echo-sound || true

echo "Deploying Echo Server in namespace: echo-sound"
cat <<EOF | kubectl -n echo-sound apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      labels:
        app: echo
    spec:
      containers:
      - name: echo
        image: gcr.io/google_containers/echoserver:1.10
        ports:
        - containerPort: 8080
EOF

echo "✅ Echo server deployment created successfully!"
```

# SOLUTION
Expose deployment as NodePort

```bash
kubectl expose deployment echo -n echo-sound --name echo-service --type NodePort --port 8080 --target-port 8080
```
```bash
kubectl get svc -n echo-sound echo-service
```
Create ingress
```bash
cat <<'EOF' > ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo
  namespace: echo-sound
spec:
  rules:
  - host: example.org
    http:
      paths:
      - path: /echo
        pathType: Prefix
        backend:
          service:
            name: echo-service
            port:
              number: 8080
EOF
```
```bash
kubectl apply -f ingress.yaml
```
```bash
kubectl get ingress -n echo-sound
```
Optional test (NodePort): curl http://<nodeIP>:<nodePort>/echo