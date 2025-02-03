# DevOps

### 1-1 Why should we run the container with a flag -e to give the environment variables?

L'utilisation des variables d'environnement permet de personnaliser une image docker sans la modifier.

### 1-2 Why do we need a volume to be attached to our postgres container?

Un volume sert à sauvegarder les données. Ces dernières étant de base stockées dans le docker, lorsqu'il s'éteint, les données sont perdues.
Avec un volume, les données sont stockées sur la machine qui host le docker.

### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

Le multistage build est utile pour faire les choses dans l'ordre, il faut que le build de l'application soit fait avant qu'elle ne soit exécutée.

```Dockerfile
# Build
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build # récupère l'image maven avec sa version spécifiée
ENV MYAPP_HOME=/opt/myapp  # Définit la variable d'environnement MYAPP_HOME.
WORKDIR $MYAPP_HOME # Change l'endoit ou on est placé dans le docker vers le chemis définit dans la variable.
COPY ./simpleapi/pom.xml . # Copie le pom.xml .
COPY ./simpleapi/src ./src # Copie le reste des fichiers nécéssaires à la compilation.
RUN mvn package -DskipTests # Run le build en ne faisant pas les tests.

# Run
FROM amazoncorretto:21 # récupère l'image pour run du java 21
ENV MYAPP_HOME=/opt/myapp  # Définit la variable d'environnement MYAPP_HOME.
WORKDIR $MYAPP_HOME # Change l'endoit ou on est placé dans le docker vers le chemis définit dans la variable.
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar # Copie le jar depuis le docker précédent vers celui-ci

ENTRYPOINT ["java", "-jar", "myapp.jar"] # Run l'application.
```

### 1-5 Why do we need a reverse proxy?

Le reverse proxy sert à faire la redirection entre le serveur web et le back.
