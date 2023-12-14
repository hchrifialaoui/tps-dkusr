# Docker / Kubernetes - TP3 : Découverte de Kubernetes
> **Objectifs du TP** :
>- Prendre en main l’environnement des TPs
>- Se familiariser avec le client kubectl
>- Découvrir les premiers concepts
>
> **Niveau de difficulté** : Débutant


## 1- Parler avec le Cluster

Nous allons simplement commencer par vérifier que le client `kubectl` fonctionne et parvient à communiquer avec le cluster K8s. Pour ce faire, nous allons simplement lancer la commande :
```sh
dev $ kubectl version --short
```
>**Questions**
>- Quelles sont les versions du client et du serveur ?
>- Les versions sont-elles strictement identiques ?


Si nous avons obtenu une réponse sans erreur, nous avons prouvé que :
- Le client est présent et correctement installé
- La configuration du client a permis de se connecter au cluster
- Les accès de l’utilisateurs permettent au moins de demander la version au cluster
- Le cluster est _a minima_ fonctionnel

## 2- Formatage des sorties de kubectl

Une des premières choses à noter avec le client `kubectl`, c’est sa capacité à adapter son format de sortie en fonction de la demande avec l’option `-o` :

```sh
dev $ kubectl version -o yaml
clientVersion:
  buildDate: "2023-09-13T09:35:49Z"
  compiler: gc
  gitCommit: 89a4ea3e1e4ddd7f7572286090359983e0387b2f
  gitTreeState: clean
  gitVersion: v1.28.2
  goVersion: go1.20.8
  major: "1"
  minor: "28"
  platform: windows/amd64
kustomizeVersion: v5.0.4-0.20230601165947-6ce0bf390ce3
serverVersion:
  buildDate: "2023-09-13T09:29:07Z"
  compiler: gc
  gitCommit: 89a4ea3e1e4ddd7f7572286090359983e0387b2f
  gitTreeState: clean
  gitVersion: v1.28.2
  goVersion: go1.20.8
  major: "1"
  minor: "28"
  platform: linux/amd64
```

Nous allons utiliser cette capacité pour produire une liste formatée des nœuds du cluster Kubernetes.

Pour commencer, voyons la forme standard de sortie de la commande qui permet d’obtenir la liste du premier type de ressources du cluster, les nœuds :

```sh
dev $ kubectl get nodes
NAME                    STATUS   ROLES    AGE     VERSION
NAME                                             STATUS   ROLES    AGE     VERSION
scw-k8s-training-dkusr-default-0d7c23665c474fb   Ready    <none>   6m37s   v1.28.2
scw-k8s-training-dkusr-default-36cd866d71df43d   Ready    <none>   6m51s   v1.28.2
scw-k8s-training-dkusr-default-650f6177dad3485   Ready    <none>   6m41s   v1.28.2
```

Cette sortie est simple et peut être adaptée pour fournir plus d’informations. Nous allons essayer à tour de rôles les commandes :
```sh
dev $ kubectl get nodes -o wide
[...]
dev $ kubectl get nodes -o json
[...]
dev $ kubectl get nodes -o yaml
[...]
```

>**Questions**
>- Combien de CPU a le nœud **n1** ?
>- Depuis combien de temps le nœud **n2** est-il enregistré ?
>- Quelle est l’adresse IP interne du nœud **n3** ?

## 3- Formatage avancé des sorties de kubectl

Nous avons vu les formats de sortie fixes, voyons désormais dans quelle mesure il est possible de les personnaliser. Pour cela, d’autres types de sortie que JSON, YAML et `wide` existent.

Les sorties JSON et YAML fournissent directement une structure de données de type `List` qui peut contenir plusieurs éléments, en fonction du nombre de nœuds sur le cluster :
```sh
dev $ kubectl get nodes -o go-template --template='{{ .kind }}{{"\n"}}'
List
dev $ kubectl get nodes -o=template --template='{{ len .items }}{{"\n"}}'
4
```
_A contrario_, si l’on demande une ressource en particulier, son type est directement récupérable :
```sh
dev $ kubectl get no/scw-k8s-training-dkusr-default-0d7c23665c474fb -o go-template --template='{{ .kind }}{{"\n"}}'
Node
```

Il est très simple de récupérer quelques champs spécifiques de la ressource :
```sh
dev $ kubectl get no/scw-k8s-training-dkusr-default-0d7c23665c474fb -o go-template --template='ce {{ .kind }} porte le nom de {{ .metadata.name }}{{"\n"}}'
ce Node porte le nom de scw-k8s-training-dkusr-default-0d7c23665c474fb
```

Pour avoir une vue sous forme de tableau personnalisée de tous les éléments, on peut utiliser la sortie **`custom-columns`**, qui reste relativement simple :
```sh
dev $ kubectl get nodes -o=custom-columns=NAME:.metadata.name,RAM:.status.capacity.memory
NAME                    RAM
scw-k8s-training-dkusr-default-0d7c23665c474fb   4045068Ki
scw-k8s-training-dkusr-default-36cd866d71df43d   4045068Ki
scw-k8s-training-dkusr-default-650f6177dad3485   4045068Ki
```

> **Exercice**
> En utilisant le format de sortie **`-o=custom-columns`**, produire la liste des nœuds du cluster avec le format suivant :
> ```
> NAME                    ARCH      KERNEL
> scw-k8s-training-dkusr-default-0d7c23665c474fb   amd64     6.2.0-36-generic
> scw-k8s-training-dkusr-default-36cd866d71df43d   amd64     6.2.0-36-generic
> scw-k8s-training-dkusr-default-650f6177dad3485   amd64     6.2.0-36-generic
> ```
