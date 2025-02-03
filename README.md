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

# Build
docker build -t mypostgres .

# Run the PostgreSQL Container with Volume

docker run -p 5432:5432 --net=app-network --name mypostgres -v data:/var/lib/postgresql/data mypostgres

# Run Adminer
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer


### Answer to Question

**1-1 Why should we run the container with a flag -e to give the environment variables?**

Using the `-e` flag to pass environment variables at runtime is more secure than hardcoding them in the Dockerfile. This approach prevents sensitive information, such as passwords, from being stored in plain text within the Dockerfile, which could be exposed if the Dockerfile is shared or stored in version control.

COPY ./initdb /docker-entrypoint-initdb.d/ dans Dockerfile

initdb folder created with sql, build and run again , it works.

# Verify Data Persistence
```sh
docker exec -it mypostgres psql -U usr -d db -c "SELECT * FROM departments;"
```

**1-2 Why do we need a volume to be attached to our postgres container?**

Attaching a volume to the PostgreSQL container ensures that the database data is stored persistently on the host machine. This way, even if the container is destroyed or recreated, the data remains intact and can be reused by the new container instance.



