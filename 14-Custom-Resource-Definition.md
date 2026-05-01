# CRD (Custom Resource Definition)


Un **CRD (Custom Resource Definition)** permet d’ajouter un **nouveau type de ressource** dans Kubernetes.

Par défaut Kubernetes connaît :  

* Pod
* Deployment
* Service

Avec un CRD, on peut créer un nouveau type, par exemple :

```text
Book
```
#### 1. Créer un CRD

```yaml id="yaml1"
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.example.com
spec:
  group: example.com
  names:
    kind: Book
    plural: books
    singular: book
    shortNames:
    - bk
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              title:
                type: string
              author:
                type: string
```
Appliquer :

```bash
kubectl apply -f book-crd.yaml
```
Ajoute un nouveau type `Book` dans Kubernetes

---

#### 2. Vérifier le CRD

```bash
kubectl get crds
```

On doit voir `books.example.com`

---

#### 3. Inspecter le CRD

```bash
kubectl describe crd books.example.com
```

Permet de comprendre :

* le **Kind** → Book
* les champs → title, author

#### 4. Créer un objet (CR)

```yaml 
apiVersion: example.com/v1
kind: Book
metadata:
  name: my-book
spec:
  title: Kubernetes Guide
  author: John Doe
```

```bash 
kubectl apply -f book.yaml
```
Crée un objet basé sur le CRD

#### 5. Lister les objets

```bash 
kubectl get books
```
Affiche les objets Book


#### 6. Voir les détails

```bash id="cmd6"
kubectl get book my-book -o yaml
```
Voir toutes les données

#### 7. Extraire un champ

```bash 
kubectl get books -o jsonpath='{.items[*].spec.title}'
```

Récupère uniquement les titres


# QUESTION
List all custom crd from cert manager and store it in custom-crd.txt.
Get the subject field from the cert manager and store it in cert-manager-subject.txt.  
  
Étape 0: Préparer l'environnement

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl create namespace cert-test

vim issuer.yml

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: test-issuer
  namespace: cert-test
spec:
  selfSigned: {}

kubectl apply -f issuer.yaml


vim cert1.yml

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: cert-one
  namespace: cert-test
spec:
  secretName: cert-one-tls
  issuerRef:
    name: test-issuer
  commonName: example.com
  subject:
    organizations:
      - dev-team
    countries:
      - FR

vim cert2.yml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: cert-two
  namespace: cert-test
spec:
  secretName: cert-two-tls
  issuerRef:
    name: test-issuer
  commonName: example.org
  subject:
    organizations:
      - ops-team
    countries:
      - US

kubectl apply -f cert1.yaml
kubectl apply -f cert2.yaml
```

# SOLUTION

Étape 1: Lister les CRDs cert-manager

```bash
kubectl get crd | grep cert-manager > custom-crd.txt
```

Étape 2: Vérifier
```bash
cat custom-crd.txt
```

Étape 3: Extraire le champ subject

```bash
kubectl get certificates --all-namespaces -o jsonpath='{.items[*].spec.subject}'
```
Pour parcourir la liste on utilise range

```bash
kubectl get certificates --all-namespaces -o jsonpath='{range .items[*]}{.spec.subject}{"\n"}{end}' > cert-manager-subject.txt
```

```bash
cat cert-manager-subject.txt
```