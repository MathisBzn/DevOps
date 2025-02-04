# DevOps

## TP Docker

### 1-1 Why should we run the container with a flag -e to give the environment variables?

L'utilisation des variables d'environnement permet de personnaliser une image docker sans la modifier.

### 1-2 Why do we need a volume to be attached to our postgres container?

Un volume sert à sauvegarder les données. Ces dernières étant de base stockées dans le docker, lorsqu'il s'éteint, les données sont perdues.
Avec un volume, les données sont stockées sur la machine qui host le docker.

### 1-3 Document your database container essentials: commands and Dockerfile.

`docker build -t <nom/nomImage> .` build l'image docker à partir du Dockerfile qui se trouve dans le répertoire courrant.
`docker run -d -p 80:80 -v /data:/data --name <nom> --network <nom du network> <nom de l'image>` run l'image en dernier paramètre en mode détaché, avec une redirection de port, des volumes, un nom customisé et une connexion à un network.
`docker ps` liste les containers actifs.

### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

Le multistage build est utile pour faire les choses dans l'ordre, il faut que le build de l'application soit fait avant qu'elle ne soit exécutée.

```Dockerfile
# Build
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build # récupère l image maven avec sa version spécifiée
ENV MYAPP_HOME=/opt/myapp  # Définit la variable d environnement MYAPP_HOME.
WORKDIR $MYAPP_HOME # Change l endoit ou on est placé dans le docker vers le chemis définit dans la variable.
COPY ./simpleapi/pom.xml . # Copie le pom.xml .
COPY ./simpleapi/src ./src # Copie le reste des fichiers nécéssaires à la compilation.
RUN mvn package -DskipTests # Run le build en ne faisant pas les tests.

# Run
FROM amazoncorretto:21 # récupère l image pour run du java 21
ENV MYAPP_HOME=/opt/myapp  # Définit la variable d environnement MYAPP_HOME.
WORKDIR $MYAPP_HOME # Change l endoit ou on est placé dans le docker vers le chemis définit dans la variable.
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar # Copie le jar depuis le docker précédent vers celui-ci

ENTRYPOINT ["java", "-jar", "myapp.jar"] # Run l application.
```

### 1-5 Why do we need a reverse proxy?

Le reverse proxy sert à faire la redirection entre le serveur web et le back.

### 1-6 Why is docker-compose so important?

Docker compose permet de faciliter la configuration et le lancement de plusieurs docker ensemble.
Il est très important pour déployer des applications fonctionnant avec plusieurs services.

### 1-7 Document docker-compose most important commands.

`docker compose up -d --build` lance le docker compose en mode détaché et en rebuildant les containers
`docker ps` liste les containers actifs.

### 1-8 Document your docker-compose file.

```Dockerfile
services:
    backend: # Nom du service back
        build: ./java # Emplacement du Dockerfile pour build
        container_name: backendTP1 # Nom du container
        networks: # Les réseaux dans lequel le docker peut interragir
            - proxy-back
            - back-bdd
        env_file: ".env" # Lien vers le fichier .env qui contient les variables d'environement
        depends_on: # le nom du Docker qui doit être build avant celui-ci
            - database

    database:
        build: ./postgres
        container_name: dbTP1
        networks:
            - back-bdd
        env_file: ".env"
        volumes:
            - ./data:/var/lib/postgresql/data

    httpd: 
        build: ./server
        container_name: proxyTP1
        ports:
            - 80:80
        networks:
            - proxy-back
        depends_on:
            - backend

networks: # Création des différents réseaux qui vont être utilisé.
    proxy-back:
    back-bdd:


```
### 1-9 Document your publication commands and published images in dockerhub.

`docker tag` Pour nommer correctement et versionner l'image.
`docker push` Pour envoyer l'image sur le repo.

### 1-10 Why do we put our images into an online repo?

Les images qui sont en lignes sont utilisables sans avoir les fichiers qui ont build l'image.
Tout le monde peut également profiter de l'utilisation des images présentes sur un repo online.

## TP Github Actions

### 2-1 What are testcontainers?

Testcontainers est une librairie qui permet de simuler des instances légères de ce qui doit être run dans un docker

