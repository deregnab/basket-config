# Basket config

## BasketBack
Utilisation d'un docker pour chaque technos

Technos:
- docker mysql
- docker compilation (maven)
- docker execution (openjdk-8)

### Network
Creation d'un reseau "basket-net" pour relier les dockers

```sh
$ sudo docker network create --driver bridge basket-net 
```

### Docker MySql:
Image mysql (ubuntu), creation d'une bdd pour l'app et son utilisateur
Redirection du port 3306 pour communiquer avec la bdd de l'exterieur

```Dockerfile
FROM mysql

ENV MYSQL_ROOT_PASSWORD root
ENV MYSQL_DATABASE basketbdd
ENV MYSQL_USER ludo
ENV MYSQL_PASSWORD ludo
ENV MYSQL_ROOT_HOST %
```

```sh
$ sudo docker build -t basket-mysql .
$ sudo docker run --name basket-mysql --network basket-net -d -p 3306:3306 basket-mysql
``` 

### Docker Maven
(Docker optionnel)
Compilation des sources dans un docker.

```sh
$ sudo docker run -it --rm --name mvn -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3.3-jdk-8 mvn clean install
```

### Docker openjdk
Image openjdk-8 (ubuntu
Execution du jar (connexion à la base via le network)
Redirection du port 8080 pour communiquer avec l'api

```sh
$ sudo docker run -d --network basket-net --name basketback -p 8080:8080 -v "$(pwd)"/target:/app -w /app openjdk:8 java -jar gs-rest-service-0.1.0.jar
```

### Notes
Sur la VM, 127.0.0.1 à la place de localhost pour utiliser la redirection des dockers
Utilisation des networks a la place du link (necessite d'utiliser le nom du container de la BDD à partir du container openjdk)

```sh
$ curl 127.0.0.1:8080
```

## BasketFront
Utilisation d'un docker angular
Redirection du port 4200 vers le port 80 sur la machine hote

```sh
$ sudo docker container run --name basketfront -v $(pwd):/opt/app -w /opt/app -d -p 80:4200 teracy/angular-cli ng serve --host=0.0.0.0
```
