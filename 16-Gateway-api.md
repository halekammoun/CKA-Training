# Gateway API

La Gateway API a été introduite pour standardiser et améliorer les enseignements tirés d’Ingress et des frameworks de service mesh comme Istio, Contour et Linkerd, qui ont montré le besoin de capacités de gestion du trafic plus avancées que ce qu’Ingress pouvait offrir.

En tant qu’alternative plus expressive et extensible à la ressource Ingress traditionnelle, la Gateway API propose :
- un design orienté rôles  
- le support de plusieurs protocoles (pas seulement HTTP/HTTPS)  
- des fonctionnalités avancées de routage  

La Gateway API est le successeur d’Ingress et devient de plus en plus importante pour gérer le trafic externe.

<p align="center">
  <img src="image.png" alt="image" width="700"/>
</p>

#### Migration Ingress → Gateway API

Remplacer :
- Ingress → Gateway + HTTPRoute  

Avantages :
- séparation claire des responsabilités  
- plus flexible  
- plus puissant  

#### Ressources Gateway API

La Gateway API introduit plusieurs objets :
<p align="center">
  <img src="image2.png" alt="image2" width="700"/>
</p>

#### 1. Gateway
définit l’infrastructure (ex: load balancer)

#### 2. GatewayClass
définit le type de controller utilisé  

#### 3. HTTPRoute / GRPCRoute
règles de routage vers les services  

#### 4. ReferenceGrant
permet le routage entre namespaces  


#### Installer les CRDs Gateway API

Par défaut, la Gateway API n’est pas installée.
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.3.0/standard-install.yaml
```
Installe les CRDs  
```bash
kubectl get crds | grep gateway.networking.k8s.io
```
Liste :
- gatewayclasses  
- gateways  
- httproutes  
- grpcroutes  
- referencegrants  

#### Installer un Gateway Controller

La Gateway API nécessite un controller.

Exemple avec Envoy :
```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.4.2 \
-n envoy-gateway-system --create-namespace
```
Installe le controller  

```bash
kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway \
--for=condition=Available
```
Attend que le controller soit prêt  

#### Créer une GatewayClass
```bash
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```
```bash
kubectl apply -f gateway-class.yaml
```
Définit quel controller sera utilisé  
```bash
kubectl get gatewayclasses
```

#### Créer un Gateway
```bash
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: hello-world-gateway
spec:
  gatewayClassName: envoy
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```
```bash
kubectl apply -f gateway.yaml
```
Ouvre un point d’entrée HTTP  

```bash
kubectl get gateways
```

#### Créer un HTTPRoute
```bash
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: hello-world-httproute
spec:
  parentRefs:
  - name: hello-world-gateway
  hostnames:
  - "hello-world.exposed"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web
      port: 3000
```
```bash
kubectl apply -f httproute.yaml
```
Définit le routage vers un Service  

```bash
kubectl get httproutes
```

#### Accéder au Gateway

Récupérer le service Envoy :
```bash
export ENVOY_SERVICE=$(kubectl get svc -n envoy-gateway-system \
--selector=gateway.envoyproxy.io/owning-gateway-name=hello-world-gateway \
-o jsonpath='{.items[0].metadata.name}')
```
#### Port forward
```bash
kubectl -n envoy-gateway-system port-forward service/${ENVOY_SERVICE} 8889:80
```

#### Tester
```bash
curl hello-world.exposed:8889
```
Résultat :
Hello World


# QUESTION 13
Migrate an existing web application from Ingress to Gateway API. You must maintain HTTPS access.
First, create a Gateway named web-gateway with hostname gateway.web.k8s.local that maintains the existing TLS and listener configuration from the existing ingress resource named web.
Next, create an HTTPRoute named web-route with hostname gateway.web.k8s.local that maintains the existing routing rules from the current Ingress resource named web.
Note - A GatewayClass named nginx is installed in the cluster. -->