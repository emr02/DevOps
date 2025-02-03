## DevOps

# TP1

## Dockerfile (Question 1-1) 

docker build -t mypostgres .

docker run -p 5432:5432 --net=app-network --name mypostgres mypostgres

docker run -p "8090:8080" --net=app-network --name=adminer -d adminer

Using the -e flag to pass environment variables at runtime is more secure because it prevents sensitive information, like passwords, from being stored in the Dockerfile. This way, the credentials are not exposed if the Dockerfile is shared or stored in version control.
