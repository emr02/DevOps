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

## Backend API

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
Le build contient le JDK nécessaire pour compiler l'application avec Maven
Le run contient uniquement le JRE, suffisant pour éxecuter l'application déjà compilée, rendant l'image plus légère.
La première étape génère les artefacts nécessaires, tandis que la seconde étape ne copie que les artefacts essentiels dans l'image finale, l'image finale sera plus petite et performante.


**1-4 Why do we need a multistage build? And explain each step of this Dockerfile.**

Multistage Build: A multistage build allows us to use multiple FROM statements in a single Dockerfile. Each FROM statement starts a new stage, and we can selectively copy artifacts from one stage to another. This approach helps in creating smaller and more efficient Docker images by including only the necessary components in the final image.

```sh
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build: This stage uses a Maven image with Amazon Corretto JDK 21 to build the application. The AS myapp-build part names this stage myapp-build.
ENV MYAPP_HOME=/opt/myapp: Sets an environment variable MYAPP_HOME to /opt/myapp.
WORKDIR $MYAPP_HOME: Sets the working directory to /opt/myapp.
COPY [pom.xml](http://_vscodecontentref_/0) .: Copies the pom.xml file to the working directory.
COPY src ./src: Copies the src directory to the working directory.
RUN mvn package -DskipTests: Runs the Maven package goal to build the application, skipping tests.

FROM amazoncorretto:21: This stage uses a smaller Amazon Corretto JRE 21 image to run the application.
ENV MYAPP_HOME=/opt/myapp: Sets an environment variable MYAPP_HOME to /opt/myapp.
WORKDIR $MYAPP_HOME: Sets the working directory to /opt/myapp.
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar: Copies the built JAR file from the myapp-build stage to the working directory.
ENTRYPOINT ["java", "-jar", "myapp.jar"]: Sets the entry point to run the JAR file using the java -jar command.
```

docker build -t simpleapi .
docker run -p 8080:8080 --name simpleapi simpleapi

docker build -t simple-api-student .
docker run -p 8080:8080 --net=app-network --name simple-api-student simple-api-student

docker build -t simple-http-server .
docker run -p 80:80 --name simple-http-server -d simple-http-server

docker stats simple-http-server
docker inspect simple-http-server
docker logs simple-http-server

docker cp simple-http-server:/usr/local/apache2/conf/httpd.conf ./httpd.conf

Benefits of Using docker cp to Retrieve httpd.conf
Customization: You can modify the default configuration to add or change settings according to your requirements.
Consistency: Ensures that you start with the default configuration provided by the image, which is known to work, and then make incremental changes.
Version Control: You can keep the customized configuration file in version control along with your Dockerfile and other project files.


docker build -t simple-http-server .
docker run -p 80:80 --net=app-network --name simple-http-server -d simple-http-server


