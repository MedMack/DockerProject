# Projet Docker
Il s'agit d'un projet universitaire de Cloud Computing en utilisant la nouvelle technologie prometteuse de [Docker](https:\\docker.com).
Dans ce tutoriel, nous allons utiliser les différents technologies de Docker : `Docker Compose`, `Docker Swarm` et `Dockerfile` pour mettre en place une architecture qui utilise deux noœuds Wordpress qui communiquent avec une bd dans un noœud/host distant.

Ce schéma vous expliquera ce qu'on va réaliser :
//image here

> À noter que ce tutoriel a été réalisé sur Mac OS X (10.9.3)

## Pré-requis
### Docker Machine 0.3.0 (ou plus)
Verification de la version par cette commande 
```
$ docker-machine -v
docker-machine version 0.3.0 (0a251fe)
```
Avoir la dernière version
```
$ curl -L https://github.com/docker/machine/releases/download/v0.3.0/docker-machine_darwin-amd64 > /usr/local/bin/docker-machine
```

### Docker Compose 1.3.0 (ou plus)

Avoir la dernière version
```
$ curl -L https://github.com/docker/compose/releases/download/1.3.0/docker-compose-`uname -s`-`uname -m` &gt; /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

Vérifier la version
```
$ docker-compose -v
docker-compose version: 1.3.0
```

### Docker Swarm 
Swarm fonctionne en tant qu'un conteneur Docker, télécharger l'image ainsi 
```
$ docker pull swarm
```

## Création du Docker Swarm Cluster
Le Swarm cluster va englober les noœuds qu'on va créer et utiliser plus tard. La création du cluster retourne un ID qui est important pour la création des noœuds.

```
docker run swarm create
Unable to find image 'swarm:latest' locally
latest: Pulling from library/swarm
d681c900c6e3: Pull complete 
188de6f24f3f: Pull complete 
90b2ffb8d338: Pull complete 
237af4efea94: Pull complete 
3b3fc6f62107: Pull complete 
7e6c9135b308: Pull complete 
986340ab62f0: Pull complete 
a9975e2cc0a3: Pull complete 
Digest: sha256:c21fd414b0488637b1f05f13a59b032a3f9da5d818d31da1a4ca98a84c0c781b
Status: Downloaded newer image for swarm:latest
5438f046c14733d64b72785de46fb167
```

Dans ce cas le identifiant du cluster est `5438f046c14733d64b72785de46fb167`. On passe à la création du Swarm master, il ne faut pas remplacer `Cluster_ID` par la valeur retournée sur votre terminal:
```
docker-machine create -d virtualbox --swarm --swarm-master --swarm-discovery token://Cluster_ID swarm-master
```
