# Projet Docker
Il s'agit d'un projet universitaire de Cloud Computing en utilisant la nouvelle technologie prometteuse de [Docker](https:\\docker.com).
Dans ce tutoriel, nous allons utiliser les différents technologies de Docker : `Docker Compose` et `Docker Swarm` pour mettre en place une architecture qui utilise deux noœuds Wordpress qui communiquent avec une bd dans un noœud/host distant.

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
$ docker run swarm create
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
$ docker-machine create -d virtualbox --swarm --swarm-master --swarm-discovery token://Cluster_ID swarm-master
```

Après la création, il faut se déplace à la hote master :
```
eval "$(docker-machine env swarm-master)"
```

On passe maintenant à la création des 3 noœuds :
```
$ docker-machine create -d virtualbox --swarm --swarm-discovery token://5438f046c14733d64b72785de46fb167 swarm-node-01
$ docker-machine create -d virtualbox --swarm --swarm-discovery token://5438f046c14733d64b72785de46fb167 swarm-node-02
$ docker-machine create -d virtualbox --swarm --swarm-discovery token://5438f046c14733d64b72785de46fb167 swarm-node-03
```

Pour voir les machines qu'on a créé, on utilise la commande
```
$ docker-machine ls
NAME            ACTIVE   DRIVER       STATE     URL                         SWARM                   DOCKER   ERRORS
default         -        virtualbox   Running   tcp://192.168.99.100:2376                           v1.9.1   
swarm-master    *        virtualbox   Running   tcp://192.168.99.101:2376   swarm-master (master)   v1.9.1   
swarm-node-01   -        virtualbox   Running   tcp://192.168.99.102:2376   swarm-master            v1.9.1   
swarm-node-02   -        virtualbox   Running   tcp://192.168.99.103:2376   swarm-master            v1.9.1   
swarm-node-03   -        virtualbox   Running   tcp://192.168.99.104:2376   swarm-master            v1.9.1
```
On remarque que dans la liste, les 4 composants `swarm-master` `swarm-node-01` `swarm-node-02` `swarm-node-03` on la même valeur du champs "SWARM", cela implique qu'ils appartiennent tous au même cluster. La hote `default` était une machine crée avant le swarm master et elle ne contient pas au cluseter.

## Création des containers

### Le noœud de données

On choisi un noœud pour les données, dans ce cas on utilisera le `swarm-node-02` , on se déplace dans le noœud :
```
eval "$(docker-machine env swarm-node-02)"
```
Et on commence la création d'un container `mysql` avec ces informations :
```
docker run -d --name db -e MYSQL_ROOT_PASSWORD=root \
> -e MYSQL_USER=mycloud -e MYSQL_PASSWORD=mycloud \
> -e MYSQL_DATABASE=wordpress mysql
```
Il est temps maintenant pour utiliser `mysql_ambassador` qui va exposer les ports du container `mysql` pour les autres noœuds en mentionnant l'adresse ip du noœud (192.168.99.103) avec le port mysql (3360):

```
docker run -d --link db:db --name mysql_ambassador \
> -p 3306:3306 svendowideit/ambassador
```

### Les noœuds Wordpress

On se déplace maintenant au premier noœud pour créer le container wordpress qui va utiliser les données du noœud 2 :

``` 
eval "$(docker-machine env swarm-node-01)"
```

La création du container Wordpress et du container mysql_ambassador (qui va relier le même container dans le noœud 2) se font avec le fichier docker-compose.yml :

```
$ docker compose up -d
```

On fait du même pour le noœud 3 :

```
$ eval "$(docker-machine env swarm-node-03)"

$ docker compose up -d
```

### Le test

Le résultat sera affiché sur l'une des IP des noœuds 1 ou 3. Sur le navigateur, `192.168.99.102` et `192.168.99.104` donneront le même résultat. Une fois l'installation du Wordpress est terminée, on peut créer des pages de n'importe quelle hote et le résultat sera le même parce qu'on utilise une seule base de données.

## Teste de la haute disponibilité

Ce n'est pas une bonne pratique mais avec cette simple architecture, si on arrête le noœud 1 ou 3 le site Wordpress restera en fonction sur l'autre hote. Pour la haute disponibilité il faut des méchanismes automatiques pour suivre le fonctionnement des services comme `Heartbeat` et des technologies qui permettent le basculement au autres containers ou noœuds qui sont en marche comme le reverse proxy ou `HAproxy`.

Toute contribution est la bienvenue.
