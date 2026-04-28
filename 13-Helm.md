# Helm et Kustomize

Les objets Kubernetes peuvent être créés, modifiés et supprimés en utilisant des commandes impératives `kubectl` ou en exécutant une commande `kubectl` sur un fichier manifest décrivant l’état souhaité d’un objet, appelé **manifest**. 

Le langage principal pour définir un manifest est **YAML**, bien que JSON soit aussi possible (moins utilisé). Il est recommandé que les équipes versionnent ces fichiers dans des dépôts afin de suivre et auditer les changements dans le temps. 

Modéliser une application Kubernetes nécessite souvent plusieurs objets :

* Deployment
* ConfigMap
* Service

Gérer tout ça avec `kubectl` uniquement est difficile → Helm & Kustomize simplifient cela.


Helm est un **gestionnaire de paquets Kubernetes** avec templating.


#### Ajouter un repository

```bash
helm repo list
```

Liste les repositories Helm déjà configurés

```bash 
helm repo add jenkinsci https://charts.jenkins.io/
```

Ajoute le repo Jenkins pour pouvoir installer ses charts

```bash 
helm repo list
```
Vérifie que le repo a bien été ajouté


#### Rechercher un chart

```bash 
helm search repo jenkinsci
```

Cherche les charts disponibles dans le repo Jenkins



#### Installer un chart

```bash 
helm install my-jenkins jenkinsci/jenkins --version 5.8.25
```

Installe Jenkins dans le cluster
Crée automatiquement Pods, Services, StatefulSets

```bash 
kubectl get all
```
Vérifie toutes les ressources créées


#### Voir les valeurs par défaut

```bash
helm show values jenkinsci/jenkins
```

Affiche les variables configurables du chart


#### Personnaliser

```bash
helm install my-jenkins jenkinsci/jenkins --version 4.6.4 \
--set controller.adminUser=boss \
--set controller.adminPassword=password \
-n jenkins --create-namespace
```

Installe Jenkins avec config personnalisée  
Crée le namespace `jenkins` si nécessaire


#### Lister les charts

```bash
helm list --all-namespaces
```

Liste tous les charts installés

#### Mettre à jour

```bash 
helm repo update
```

Met à jour la liste des charts disponibles

```bash
helm upgrade my-jenkins jenkinsci/jenkins --version 5.8.26
```

Met à jour Jenkins vers une nouvelle version

#### Désinstaller

```bash
helm uninstall my-jenkins
```
Supprime le chart + toutes ses ressources

# QUESTION

Install ArgoCD in the cluster by performing the following tasks: Add the official Argo CD Helm repository with the name argo.
Generate a template of the ArgoCD Helm Chart version 7.7.3 for the argocd namespace and save it to ~/argo-helm.yaml. Configure the chart to not install CRDs.
Note - The Argo CD CRDs have already been pre-installed in the cluster.

# CORRECTION

1. Ajouter le repository Helm
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```
2. Vérifier les valeurs disponibles du chart (IMPORTANT)

```bash
helm show values argo/argo-cd --version 7.7.3 | grep -i crd -A2
```

Résultat typique :
```yaml
crds:
  install: true
```

Donc pour ne PAS installer les CRDs :
```bash
--set crds.install=false
```

3. Créer le namespace
```bash
kubectl create namespace argocd
```

4. Générer le template Helm
```bash
helm template argocd argo/argo-cd --version 7.7.3 --namespace argocd --set crds.install=false > ~/argo-helm.yaml
```
Vérification
```bash
ls -l ~/argo-helm.yaml
```
