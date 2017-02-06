# NetBeansDay2017
NetBeans Day France 2017 - Workshop @Grenoble

## Prérequis du workshop
- JDK 8: [Lien téléchargement](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 
- NetBeans IDE 8.2: [Lien téléchargement](https://netbeans.org/downloads/)
- SpringBoot plugin for NetBeans by [Alex Falappa](https://github.com/AlexFalappa): [Lien téléchargement](https://github.com/AlexFalappa/nb-springboot)
- Docker: [Lien téléchargement](https://www.docker.com/products/docker-toolbox)
- Git et un compte Github

## Contenu du workshop
### Introduction à Spring Boot
Introduction des bases de développement Spring Boot:
1. Génération de projet via Spring Initializr
2. Présentation du SpringBoot & autoconfigurations
3. Création d'une application CRUD: Spring Data JPA + Spring Data REST
4. Présentation de Spring Actuator

### Introduction à Docker
1. Présentation de Docker, de la virtualisation et du packaging Docker Containers
2. Présentation de la terminologie Docker: Image; Container; Dockerfile; Docker-Machine  
3. Présentation de Docker-Compose, Docker-Swarm
4. Présentation de Docker Hub et de la livraison continue via Docker

## Code Snippets

* application.properties
```properties
management.security.enabled=false
spring.jpa.hibernate.ddl-auto=create

spring.datasource.url=jdbc:postgresql://postgres:5432/demo
spring.datasource.platform=POSTGRESQL
spring.datasource.username=developer
spring.datasource.password=p4SSW0rd
```

* PostgreSQL Maven Dependency
```xml
<dependency>
    <groupId>postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>9.1-901-1.jdbc4</version>
</dependency>
```

* Dockerfile example
```dockerfile
FROM openjdk:8-jre-alpine

ADD DemoApp.jar /app.jar

EXPOSE 8000 8080

CMD java -Djava.security.egd=file:/dev/./urandom -jar /app.jar
```


* Spotify Docker Maven Plugin
```xml
<plugin>
   <groupId>com.spotify</groupId>
   <artifactId>docker-maven-plugin</artifactId>
   <version>VERSION</version>
   <configuration>
      <imageName>demo-app</imageName>
      <dockerDirectory>docker-folder</dockerDirectory>
      <resources>
         <resource>
            <targetPath>/</targetPath>
            <directory>${project.build.directory}</directory>
            <include>${project.build.finalName}.jar</include>
         </resource>
      </resources>
   </configuration>
</plugin>
```
* Build Docker Image using Maven
```zsh
mvn clean install docker:build
```

* Docker Run PostgreSQL Container
```docker
docker run -d \                                                                       
    --name demo-postgres \
    -e POSTGRES_USER=developer
    -e POSTGRES_PASSWORD=p4SSW0rd \
    -e POSTGRES_DB=demo \
    -p 5432:5432 postgres:latest
```

* Docker Run App Container
```docker
docker run -d \
    --name app-container \
    --link demo-postgres:postgres
    -p 8080:8080
    -p 8000:8000
    demo-app
```

* Docker Run PostgreSQL Container with persistent data
```docker
docker run -d -p 3306:3306 -v /path/in/host:/var/lib/postgresql/data postgres:latest
```

* Docker-compose file
```yaml
version: '2'
services:
    demoapp-app:
        image: demo-app
        external_links:
            - demoapp-postgres:postgres
        ports:
            - 8080:8080
            - 8000:8000
    demoapp-postgres:
        image: postgres
        environment:
            - POSTGRES_DB=demo
            - POSTGRES_PASSWORD=p4SSW0rd
            - POSTGRES_USER=developer
        ports:
            - 5432:5432
```

### Deboguer une application déployée dans un conteneur Docker
En se basant sur [ce chapitre de la doc officielle](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-running-your-application.html):
Pour démarrer l'application en mode Debug; il suffit d'ajouter ces paramètres dans la ligne d'exécution:
```bash
-Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n
```

Donc le nouveau Dockerfile sera:
```dockerfile
FROM openjdk:8-jre-alpine

ADD DemoApp.jar /app.jar

EXPOSE 8000 8080

CMD java -Djava.security.egd=file:/dev/./urandom -jar -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n /app.jar
```

Ensuite, brancher NetBeans IDE sur le port 8000 qu'on a designé pour l'écoute du mode débogage:

#### Configure Remote Debugging

Dans le menu cliquer `Debug` > `Attach Debugger...`

![](https://raw.githubusercontent.com/docker/labs/master/developer-tools/java-debugging/images/netbeans_debug_attach_debugger_menu.png)

Vérifier que le port est 8000, ensuite cliquer `OK`.

![](https://raw.githubusercontent.com/docker/labs/master/developer-tools/java-debugging/images/netbeans_debug_attach_debugger_configure.png)

