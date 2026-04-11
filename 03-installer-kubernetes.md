# installer Kubernetes Cluster kubeadm + Flannel (RHEL 10)

Cluster :

* 1 Master (control-plane)
* 2 Workers
* CNI : Flannel
* Méthode : kubeadm

---

# 0. Prérequis

Configuration minimale recommandée :

Master :

* 2 CPU
* **3 GB RAM recommandé**
* 20 GB disk

Workers :

* 1 CPU
* 1.5 GB RAM minimum
* 15 GB disk

Toutes les machines doivent communiquer entre elles via IP.

---

# 1. Définir hostname unique

Chaque node Kubernetes doit avoir un nom unique pour éviter les conflits lors du join.

## Master

```bash
hostnamectl set-hostname master
```

Cette commande change le hostname de la machine pour que Kubernetes identifie ce node comme control-plane.

## Worker1

```bash
hostnamectl set-hostname worker1
```

Cette commande définit un nom unique pour le premier worker.

## Worker2

```bash
hostnamectl set-hostname worker2
```

Cette commande définit un nom unique pour le second worker.

---

# 2. Configurer /etc/hosts

```bash
vi /etc/hosts
```

On modifie le fichier hosts pour que chaque node puisse résoudre les noms master, worker1 et worker2.

Ajouter :

```
192.168.5.129 master
192.168.5.130 worker1
192.168.5.131 worker2
```

Tester :

```bash
ping master
```

Permet de vérifier que la résolution DNS locale fonctionne.

```bash
ping worker1
```

Teste la communication réseau vers worker1.

```bash
ping worker2
```

Teste la communication réseau vers worker2.

---

# 3. Désactiver swap

```bash
swapoff -a
```

Kubernetes nécessite que la swap soit désactivée sinon kubelet refuse de démarrer.

```bash
sed -i '/ swap / s/^/#/' /etc/fstab
```

Cette commande empêche la swap de se réactiver après reboot.

---

# 4. Charger modules kernel

```bash
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Cette commande configure les modules réseau nécessaires pour Kubernetes.

```bash
modprobe overlay
```

Charge le module overlay utilisé par containerd.

```bash
modprobe br_netfilter
```

Permet au kernel de filtrer le trafic réseau des containers.

---

# 5. Configurer sysctl

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF
```

Active le routage réseau entre pods Kubernetes.

```bash
sysctl --system
```

Applique immédiatement la configuration réseau.

---

# 6. Désactiver firewalld

```bash
systemctl disable --now firewalld
```

On désactive firewalld pour éviter le blocage du réseau entre les pods.

Alternative ouvrir ports :

```bash
firewall-cmd --permanent --add-port=6443/tcp
```

Ouvre le port API Kubernetes.

```bash
firewall-cmd --permanent --add-port=10250/tcp
```

Ouvre le port kubelet.

```bash
firewall-cmd --permanent --add-port=8472/udp
```

Ouvre le port overlay réseau Flannel.

```bash
firewall-cmd --reload
```

Recharge la configuration firewall.

---

# 7. Installer containerd

```bash
dnf install -y containerd
```

Installe container runtime utilisé par Kubernetes.

```bash
mkdir -p /etc/containerd
```

Crée dossier configuration containerd.

```bash
containerd config default | tee /etc/containerd/config.toml
```

Génère configuration par défaut.

```bash
vi /etc/containerd/config.toml
```

Permet de modifier la configuration containerd.

Changer :

```
SystemdCgroup = true
```

Active cgroup systemd pour compatibilité kubelet.

```bash
systemctl enable --now containerd
```

Démarre containerd et active au boot.

---

# 8. Installer Kubernetes

```bash
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
```

Ajoute repository Kubernetes officiel.

```bash
dnf install -y kubelet kubeadm kubectl
```

Installe les composants Kubernetes.

```bash
systemctl enable --now kubelet
```

Démarre kubelet qui gère les pods.

---

# 9. Initialiser le master

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```

Initialise le cluster Kubernetes avec CIDR réseau utilisé par Flannel.

```bash
mkdir -p $HOME/.kube
```

Crée dossier config kubectl.

```bash
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Copie config cluster pour kubectl.

```bash
chown $(id -u):$(id -g) $HOME/.kube/config
```

Donne permissions utilisateur.

```bash
kubectl get nodes
```

Vérifie que le master est enregistré dans le cluster.

---

# 10. Installer Flannel

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

Déploie le plugin réseau Flannel pour permettre communication pods.

```bash
kubectl get pods -n kube-flannel
```

Vérifie que les pods Flannel sont running.

```bash
kubectl get nodes
```

Les nodes deviennent Ready après installation réseau.

---

# 11. Joindre workers

```bash
kubeadm join 192.168.5.129:6443 ...
```

Ajoute worker au cluster Kubernetes.

---

# 12. Reset si join faux

```bash
kubeadm reset -f
```

Supprime la configuration Kubernetes du worker.

```bash
rm -rf /etc/cni/net.d
```

Supprime configuration réseau précédente.

```bash
rm -rf /var/lib/cni
```

Nettoie plugins réseau.

```bash
systemctl restart kubelet
```

Redémarre kubelet après reset.

---

# 13. Recréer join command

```bash
kubeadm token create --print-join-command
```

Génère nouvelle commande join si token expiré.

---

# Vérification finale

```bash
kubectl get nodes
```

Résultat attendu :

```
master    Ready
worker1   Ready
worker2   Ready
```
