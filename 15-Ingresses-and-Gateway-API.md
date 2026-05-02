# Ingress

Ce chapitre a exploré l’objectif et la création de la primitive **Service**. Lorsqu’il devient nécessaire d’exposer une application à des consommateurs externes, le choix du type de Service approprié devient crucial. Le choix le plus pratique consiste souvent à créer un Service de type **LoadBalancer**. Un tel Service offre des capacités de load balancing en attribuant une adresse IP externe accessible aux consommateurs en dehors du cluster Kubernetes.

Cependant, utiliser un Service de type **LoadBalancer** pour chaque application accessible de l’extérieur présente des inconvénients. Dans un environnement cloud, chaque Service entraîne la création d’un load balancer externe, ce qui augmente les coûts. De plus, gérer plusieurs objets Service peut devenir complexe, car un nouvel objet doit être créé pour chaque microservice exposé.

Pour résoudre ces problèmes, la primitive **Ingress** intervient en fournissant un point d’entrée unique et équilibré pour une application. Un Ingress est capable de router les requêtes HTTP(S) externes vers un ou plusieurs Services dans le cluster, en se basant sur un nom d’hôte DNS et un chemin d’URL.

#### Travailler avec les Ingress

Un Ingress expose des routes HTTP (et éventuellement HTTPS) vers des clients externes via une URL accessible. Les règles définies déterminent comment le trafic est routé.

Dans les environnements cloud, un load balancer externe est souvent utilisé. L’Ingress reçoit une IP publique et peut router vers plusieurs Services selon les chemins d’URL.

Exemple :
- `/app` → App Service
- `/metrics` → Metrics Service



# Installer un Ingress Controller

Un Ingress nécessite un **controller** pour fonctionner.
Ce controller applique les règles définies.

Exemple avec NGINX :

kubectl get pods -n ingress-nginx

👉 Le Pod doit être en état **Running** pour que l’Ingress fonctionne.

---

# Déployer plusieurs Ingress Controllers

On peut avoir plusieurs controllers.
Le choix se fait via :

spec.ingressClassName

Lister les classes :

kubectl get ingressclasses

👉 Kubernetes utilise une classe par défaut si aucune n’est définie.

---

# Configurer les règles Ingress

Un Ingress peut définir :
- un host
- des paths
- un backend (service + port)

---

## Table 18-1. Règles Ingress

| Type | Exemple | Description |
|------|--------|------------|
| host | next.example.com | Si défini, la règle s’applique à ce host |
| paths | /app | Le trafic doit matcher host + path |
| backend | app-service:8080 | Service cible |

---

# Créer un Ingress

### Méthode impérative

kubectl create ingress next-app \
  --rule="next.example.com/app=app-service:8080" \
  --rule="next.example.com/metrics=metrics-service:9090"

👉 Crée un Ingress avec 2 règles

---

### YAML (recommandé)

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: next-app
spec:
  rules:
  - host: next.example.com
    http:
      paths:
      - path: /app
        pathType: Exact
        backend:
          service:
            name: app-service
            port:
              number: 8080

👉 Permet une configuration plus claire

---

# 🔐 Exemple avec certificat TLS (HTTPS)

👉 Pour sécuriser un Ingress, on utilise un **Secret TLS** contenant :
- certificat
- clé privée

---

## 1. Créer un Secret TLS

kubectl create secret tls tls-secret \
  --cert=cert.pem \
  --key=key.pem

👉 Stocke le certificat dans Kubernetes

---

## 2. Ingress avec HTTPS

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

# Types de path

| Type | Description |
|------|------------|
| Exact | correspond exactement |
| Prefix | correspond au début |

👉 Différence : gestion du `/` final

---

# Lister les Ingress

kubectl get ingress

👉 Affiche :
- host
- IP
- ports

---

# Détails d’un Ingress

kubectl describe ingress next-app

👉 Permet de voir :
- règles
- erreurs
- services associés

---

# Résolution d’erreur

Si un service n’existe pas :

error: endpoints not found

👉 Solution : créer le Pod + Service

---

# Accéder à un Ingress

1. Récupérer IP :

kubectl get ingress next-app -o jsonpath="{.status.loadBalancer.ingress[0].ip}"

2. Ajouter dans `/etc/hosts`

192.168.66.4 next.example.com

---

## Tester

wget next.example.com/app

👉 fonctionne

wget next.example.com/app/

👉 peut échouer (Exact vs Prefix)

# QUESTION
Migrate an existing web application from Ingress to Gateway API. You must maintain HTTPS access.
First, create a Gateway named web-gateway with hostname gateway.web.k8s.local that maintains the existing TLS and listener configuration from the existing ingress resource named web.
Next, create an HTTPRoute named web-route with hostname gateway.web.k8s.local that maintains the existing routing rules from the current Ingress resource named web.
Note - A GatewayClass named nginx is installed in the cluster.