# Lab Vagrant + K3s

## Description

Ce lab déploie un petit cluster Kubernetes K3s sur deux machines virtuelles Debian 12 gérées par Vagrant et VirtualBox.

- `d3bs` est la VM server Kubernetes. Elle reçoit l'adresse privée `192.168.50.67` et héberge le control plane K3s.
- `d3bsw` est la VM worker. Elle reçoit l'adresse privée `192.168.50.69` et rejoint le cluster à l'aide du token partagé.
- Les deux VMs disposent chacune de 2 CPU et 2 Go de RAM.
- K3s et ses dépendances nécessaires sont installés automatiquement pendant le provisionnement.

Le fichier de configuration se trouve dans [`setup/Vagrantfile`](setup/Vagrantfile).

## Prérequis

Installer les outils suivants sur l'hôte :

- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Une connexion Internet pour télécharger la box Debian et K3s

Le réseau privé `192.168.50.0/24` ne doit pas déjà être utilisé sur la machine hôte.

## Création du cluster

Depuis le dossier `setup`, démarrer les deux machines :

```bash
cd setup
vagrant up
```

Vagrant crée successivement `d3bs`, puis `d3bsw`. Le worker attend que le serveur accepte les connexions sur le port `6443` avant de l'intégrer au cluster.

Pour vérifier le statut des machines :

```bash
vagrant status
```

## Utiliser le cluster

Se connecter au serveur :

```bash
vagrant ssh d3bs
```

Puis vérifier que le serveur et le worker sont enregistrés :

```bash
kubectl get nodes
```

Résultat attendu :

```text
NAME   STATUS   ROLES                  AGE   VERSION
 d3bs    Ready    control-plane,master   ...   v...
 d3bsw   Ready    <none>                 ...   v...
```

Lister les pods du cluster :

```bash
kubectl get pods -A
```

## Commandes utiles

Depuis le dossier `setup` :

```bash
# Se connecter au serveur
vagrant ssh d3bs

# Se connecter au worker
vagrant ssh d3bsw

# Arrêter les deux VMs
vagrant halt

# Redémarrer les VMs
vagrant up

# Rejouer le provisionnement
vagrant provision

# Supprimer entièrement les VMs
vagrant destroy -f
```

## Résolution de problèmes

Afficher les logs de provisionnement :

```bash
vagrant up --debug
```

Si le worker n'apparaît pas dans le cluster, vérifier depuis `d3bsw` que le port API du serveur est joignable, puis consulter le service K3s :

```bash
nc -zv 192.168.50.67 6443
sudo systemctl status k3s-agent
sudo journalctl -u k3s-agent -f
```

Sur le serveur, les logs du service peuvent être consultés avec :

```bash
sudo systemctl status k3s
sudo journalctl -u k3s -f
```

> Le token K3s est actuellement défini dans le `Vagrantfile`. Pour une utilisation en dehors du lab, remplacez la valeur par un token robuste et secret.
