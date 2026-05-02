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


<!-- #### Ingress avec HTTPS

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-app
spec:
  tls:
  - hosts:
    - secure.example.com
    secretName: tls-secret
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80

👉 Active HTTPS sur le domaine  
👉 Kubernetes utilise le Secret TLS  

---



# QUESTION 12 nour
# QUESTION 13 hale
Migrate an existing web application from Ingress to Gateway API. You must maintain HTTPS access.
First, create a Gateway named web-gateway with hostname gateway.web.k8s.local that maintains the existing TLS and listener configuration from the existing ingress resource named web.
Next, create an HTTPRoute named web-route with hostname gateway.web.k8s.local that maintains the existing routing rules from the current Ingress resource named web.
Note - A GatewayClass named nginx is installed in the cluster. -->