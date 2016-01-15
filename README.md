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

## Creéation du Docker Swarm Cluster
Le Swarm cluster c
