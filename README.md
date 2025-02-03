# TP1

## Database

```sh
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY ./initdb /docker-entrypoint-initdb.d/
```

- **FROM** définit l'image de base du conteneur, dans ce cas, PostgreSQL.  
- **COPY** transfère un fichier depuis la machine hôte vers un répertoire spécifique du conteneur Docker.  
- **ENV** sert à définir des variables d'environnement au sein du conteneur Docker.

```sh
# Create network, communication entre 2 conteneurs, il faut créer un réseau commun

docker network create app-network
```

```sh
# To restart admirer

docker run -d --name adminer --network app-network -p 8090:8080 adminer
```

```sh
# Build docker image

docker build -t mypostgres .
```

```sh
# Run the PostgreSQL Container with Volume

docker run -p 5432:5432 --net=app-network --name mypostgres -v data:/var/lib/postgresql/data mypostgres

-p exposer les ports du conteneur sur la machine hôte.  
--name sert à attribuer un nom au conteneur.  
-v gère les volumes pour la persistance des données, sinon si container détruit, données conservées
```
```sh
#Run Adminer

docker run -p "8090:8080" --net=app-network --name=adminer -d adminer
```

**1-1 Why should we run the container with a flag -e to give the environment variables?**

Utiliser l'option `-e` pour passer des variables d'environnement lors de l'exécution est plus sûr que de les écrire directement dans le Dockerfile. Cela évite de stocker des informations sensibles, comme des mots de passe, en clair dans le fichier, ce qui pourrait poser un risque si le Dockerfile est partagé ou enregistré dans un gestionnaire de versions.

**1-2 Why do we need a volume to be attached to our postgres container?**

Attacher un volume au conteneur PostgreSQL garantit que les données de la base de données sont stockées de manière persistante sur la machine hôte. Ainsi, même si le conteneur est détruit ou recréé, les données restent intactes et peuvent être réutilisées par la nouvelle instance du conteneur.

**1-3 Document your database container essentials: commands and Dockerfile.**

Done

## Backend API (1-4)

###  Build
```sh
# Build
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build
ENV MYAPP_HOME=/opt/myapp 
WORKDIR $MYAPP_HOME
COPY /simpleapi/pom.xml .
COPY /simpleapi/src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:21
ENV MYAPP_HOME=/opt/myapp 
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]
```
Un build multistage permet d'utiliser plusieurs instructions FROM dans un seul Dockerfile. Chaque instruction FROM démarre une nouvelle étape, et nous pouvons sélectionner les artefacts à copier d'une étape à l'autre.
Le build contient le JDK nécessaire pour compiler l'application avec Maven
Le run contient uniquement le JRE, suffisant pour éxecuter l'application déjà compilée, rendant l'image plus légère.
La première étape génère les artefacts nécessaires, tandis que la seconde étape ne copie que les artefacts essentiels dans l'image finale, l'image finale sera plus petite et performante.

### Explication du Dockerfile

#### 1. **Phase de Build**

```dockerfile
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build
```
Utilise Maven et le JDK Amazon Corretto pour compiler l'application.

```dockerfile
WORKDIR /opt/myapp
COPY /simpleapi/pom.xml .
COPY /simpleapi/src ./src
RUN mvn package -DskipTests
```
Définit le répertoire de travail, copie le code source et compile l'application en un fichier JAR.

#### 2. **Phase d'Exécution**

```dockerfile
FROM amazoncorretto:21
```
Utilise l'image JRE Amazon Corretto pour l'exécution de l'application.

```dockerfile
COPY --from=myapp-build /opt/myapp/target/*.jar /opt/myapp/myapp.jar
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```
Copie le JAR compilé et exécute l'application avec `java -jar`.

---

Cette approche sépare la construction (avec JDK) et l'exécution (avec JRE) pour une image finale plus légère et optimisée.

## Http server

**1-5 Why do we need a reverse proxy?**

Un reverse proxy est utilisé pour rediriger les requêtes des clients vers un ou plusieurs serveurs backend, en agissant comme un intermédiaire. 
Avantages : sécurité en masquant le serveur backend, gestion du SSL/TLS etc ..
Dans notre cas, il redirige les requêtes HTTP vers l'application backend de manière simple.

```sh

# database
docker build -t mypostgres .
docker run -p 5432:5432 --net=app-network --name mypostgres -v data:/var/lib/postgresql/data mypostgres

# API
docker build -t simple-api-student .
docker run -p 8080:8080 --net=app-network --name simple-api-student simple-api-student

# Server
docker build -t simple-http-server .
docker run -p 80:80 --net=app-network --name simple-http-server -d simple-http-server

# To check
docker stats simple-http-server
docker inspect simple-http-server
docker logs simple-http-server

```

`Dockerfile` :
```dockerfile
FROM httpd:2.4

COPY httpd.conf /usr/local/apache2/conf/httpd.conf
COPY ./index.html /usr/local/apache2/htdocs/
```

`httpd.conf` que on a récupéré avec : 
```sh
docker cp simple-http-server:/usr/local/apache2/conf/httpd.conf ./httpd.conf
```
```sh
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://simple-api-student:8080/
ProxyPassReverse / http://simple-api-student:8080/
</VirtualHost>
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

Avantages de l'utilisation de `docker cp` pour récupérer `httpd.conf` :

- **Personnalisation** : Ajuster la configuration selon les besoins.
- **Cohérence** : Configuration par défaut qui fonctionne, puis y apporter des modifications.
- **Contrôle de version** : Garder le fichier de configuration personnalisé, avec le Dockerfile et autres fichiers.

## Link application

**1-6 Why is docker-compose so important?**

Docker-compose est important car il permet de gérer plusieurs conteneurs Docker avec une seule commande. Il simplifie le déploiement et l'organisation d'applications utilisant plusieurs conteneurs, en facilitant le démarrage, l'arrêt et la gestion des services d'une application. Cela permet de gagner du temps et d'assurer que la configuration est fiable et peut être reproduite facilement.

---

**1-8 Document docker-compose most important commands.**

- **Démarrer les conteneurs** :  
  ```shell
  docker-compose up
  ```

- **Démarrer les conteneurs en mode détaché** (en arrière-plan) :  
  ```shell
  docker-compose up -d
  ```

- **Arrêter et supprimer les conteneurs** :  
  ```shell
  docker-compose down
  ```

- **Mettre en pause les conteneurs** :  
  ```shell
  docker-compose pause
  ```

- **Reprendre les conteneurs mis en pause** :  
  ```shell
  docker-compose unpause
  ```

- **Voir les logs des conteneurs** :  
  ```shell
  docker-compose logs
  ```

- **Construire les images** :  
  ```shell
  docker-compose build
  ```

---

**1-8 Document your docker-compose file.**

Voici un exemple de structure de fichier `docker-compose.yml` :

```yaml
version: '3.7'

services:
  backend:
    build:
      context: ./backend_api/app2
      dockerfile: Dockerfile
    container_name: simple-api-student
    networks:
      - app-network
    depends_on:
      - database

  database:
    build:
      context: ./postgres
      dockerfile: Dockerfile
    container_name: mypostgres
    networks:
      - app-network
    volumes:
      - ./data:/var/lib/postgresql/data

  httpd:
    build:
      context: ./http_server
      dockerfile: Dockerfile
    container_name: simple-http-server
    networks:
      - app-network
    ports:
      - 8040:80

networks:
  app-network:
```

#### services

1. **backend** : 
   - Construit l'image à partir de `./backend_api/app2` avec le Dockerfile.
   - Dépend de `database` et utilise le réseau `app-network`.

2. **database** : 
   - Construit l'image à partir de `./postgres`.
   - Utilise un volume pour persister les données dans `./data` et se connecte au réseau `app-network`.

3. **httpd** : 
   - Construit l'image à partir de `./http_server`.
   - Expose le port `8040` local vers le port `80` du conteneur et utilise le réseau `app-network`.

---

#### networks

```yaml
networks:
  app-network:
```
Crée un réseau `app-network` pour que tous les services puissent communiquer entre eux.

En résumé, ce fichier configure un backend, une base de données PostgreSQL et un serveur HTTP, tous connectés via un réseau privé.

---

## Publish

**1-9 Document your publication commands and published images in dockerhub.**

```shell
# Se connecter à Docker Hub :
 docker login

#Taguer votre image :
docker tag mypostgres wanoni/my-database:1.0

# Pousser l'image vers Docker Hub :
docker push wanoni/my-database:1.0
```

---

**1-10 Why do we put our images into an online repo?**

Mettre nos images dans un dépôt en ligne comme Docker Hub permet de les partager facilement avec d'autres, de les sauvegarder, d'y accéder à distance et de gérer les versions. C'est essentiel pour collaborer et déployer de manière efficace.

