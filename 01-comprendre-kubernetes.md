# Kubernetes en bref

Il est utile d’avoir une vue rapide de Kubernetes et de son fonctionnement si vous débutez.  
Ce chapitre résume les concepts les plus importants.

---

# Qu’est-ce que Kubernetes ?

Pour comprendre Kubernetes, commençons par définir microservices et containers.

Une architecture microservices consiste à développer une application sous forme de plusieurs services indépendants qui communiquent entre eux.

Si ces services sont exécutés dans des containers, il faut en gérer beaucoup tout en prenant en compte :

- la scalabilité  
- la sécurité  
- la persistance des données  
- le load balancing  

Des outils comme BuildKit ou Podman permettent de créer des images de containers.  
Des moteurs comme Docker ou containerd permettent d’exécuter ces containers.

Cela fonctionne bien sur une machine locale, mais devient difficile à gérer à grande échelle.

Kubernetes est un outil d’orchestration de containers qui permet de gérer des centaines ou milliers de containers sur :

- machines physiques  
- machines virtuelles  
- cloud  

Kubernetes gère aussi automatiquement :

- le scaling  
- la sécurité  
- la persistance  
- le load balancing  

---

# Fonctionnalités

## Modèle déclaratif

Avec Kubernetes, on ne dit pas comment faire.  
On décrit simplement l’état souhaité.

Exemple :  
on veut 3 pods  
Kubernetes maintient toujours 3 pods  

Cela se fait avec des fichiers YAML ou JSON.

---

## Autoscaling

Kubernetes peut :

- augmenter les ressources si la charge augmente  
- diminuer si la charge baisse  

Cela peut être manuel ou automatique.

---

## Gestion des applications

Kubernetes permet :

- déploiement de nouvelles versions  
- rolling update  
- rollback vers ancienne version  

---

## Stockage persistant

Les containers perdent leurs données après redémarrage lorsque les données sont stockées uniquement dans le filesystem du container.

Pour conserver les données, il est possible de monter un **stockage persistant**.  
Ce stockage est séparé du cycle de vie du container ou du Pod.

Kubernetes permet plusieurs possibilités de stockage persistant.

Ces mécanismes permettent de conserver les données même si :

- le container redémarre  
- le Pod est recréé  
- l’application est redéployée  

---

## Networking

Kubernetes permet :

- communication entre containers  
- accès externe  
- load balancing interne et externe

# Architecture kubernetes
