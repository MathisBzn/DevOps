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

### 2-2 Document your Github Actions configurations.

```yml
name: CI devops 2025 #Nom du workflow
on: #Se déclenche au moment du ...
  push: #push sur ...
    branches: #Les branches qui sont listées ci-dessous.
      - main
      - develop
  pull_request:

jobs: #Lorsqu'il se déclenche, run ces jobs
  test-backend: #Nom d'un job
    runs-on: ubuntu-22.04 #Distrib sur laquelle va run le job
    steps: #Les différentes étapes du job. Chacune doit contenir un uses ou un run
      - uses: actions/checkout@v4

      - name: Set up JDK 21 #Nom de l'action
        uses: actions/setup-java@v3
        with:
          distribution: 'corretto' #Distrib du docker sur lequel va run java
          java-version: '21'

      - name: Build and test with Maven
        run: | #run les commandes suivantes unes après les autres.
          cd java/simple-api-student-main
          mvn clean verify
```

### 2-3 For what purpose do we need to push docker images?

On a besoin de push les images pour les personnes qui veulent pull le projet sans avoir à build les images docker.
Plus simple pour que le projet soit utilisable par le plus grand nombre.

### 2-4 Document your quality gate configuration.

```yml
name: CI devops 2025 #Nom du workflow
on: #Se déclenche au moment du ...
  push: #push sur ...
    branches: #Les branches qui sont listées ci-dessous.
      - main
      - develop
  pull_request:

jobs: #Lorsqu'il se déclenche, run ces jobs
  test-backend:
    name: Build and analyze
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: 21 #Java 21
          distribution: 'corretto'
      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: Build and analyze
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: | #On se déplace dans le bon dossier et on est avec sonar
              cd java/simple-api-student-main 
              mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=MathisBzn_DevOps

  build-and-push-docker-image: #Nom d'un job
    needs: test-backend
    runs-on: ubuntu-22.04 #Distrib sur laquelle va run le job
    steps: #Les différentes étapes du job. Chacune doit contenir un uses ou un run
      - uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3 #On se log à docker hub
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./java
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-simple-api:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./postgres
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-postgres:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          context: ./server
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-httpd:latest
          push: ${{ github.ref == 'refs/heads/main' }}
``` 

## Ansible

### 3-1 Document your inventory and base commands

`ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become`
`-i` pour mettre le lien vers le fichier d'inventaire
`-m` nom du module que on veut executer
`-a` arguments du module executé

```yml
all:
 vars:
   ansible_user: admin #User sur le serveur
   ansible_ssh_private_key_file: /id_rsa #Lien vers la clé RSA
 children:
   prod: #Group d'hosts
     hosts: mathis.bozon.takima.cloud #Host pour lequel on veut config
```

### 3-2 Document your playbook

```yml
- hosts: all #Fais pour tous les hosts qui sont dans le setup.yml; On peut mettre prod dans notre cas pour focus uniquement les hosts dans le groupe prod
  gather_facts: true
  become: true #Est root
  roles: #Execute les données dans le role
    - docker #Va executer les tasks présents dans roles/docker/tasks/main.yml
```
