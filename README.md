## DevOps

# TP1

## Dockerfile (Question 1-1) 

```sh
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY ./initdb /docker-entrypoint-initdb.d/
```

## Build
docker build -t mypostgres .

## Run the PostgreSQL Container with Volume

docker run -p 5432:5432 --net=app-network --name mypostgres -v data:/var/lib/postgresql/data mypostgres

## Run Adminer
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer


### Answer to Question

**1-1 Why should we run the container with a flag -e to give the environment variables?**

Using the `-e` flag to pass environment variables at runtime is more secure than hardcoding them in the Dockerfile. This approach prevents sensitive information, such as passwords, from being stored in plain text within the Dockerfile, which could be exposed if the Dockerfile is shared or stored in version control.

COPY ./initdb /docker-entrypoint-initdb.d/ dans Dockerfile

initdb folder created with sql, build and run again , it works.

### Verify Data Persistence
```sh
docker exec -it mypostgres psql -U usr -d db -c "SELECT * FROM departments;"
```

**1-2 Why do we need a volume to be attached to our postgres container?**

Attaching a volume to the PostgreSQL container ensures that the database data is stored persistently on the host machine. This way, even if the container is destroyed or recreated, the data remains intact and can be reused by the new container instance.

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

docker run -p 8080:8080 --net=app-network --name simple-api-student simple-api-student

docker build -t simple-http-server .
docker run -p 8080:80 --name simple-http-server-container -d simple-http-server

docker stats simple-http-server
docker inspect simple-http-server
docker logs simple-http-server

docker cp simple-http-server-container:/usr/local/apache2/conf/httpd.conf ./httpd.conf

Benefits of Using docker cp to Retrieve httpd.conf
Customization: You can modify the default configuration to add or change settings according to your requirements.
Consistency: Ensures that you start with the default configuration provided by the image, which is known to work, and then make incremental changes.
Version Control: You can keep the customized configuration file in version control along with your Dockerfile and other project files.



