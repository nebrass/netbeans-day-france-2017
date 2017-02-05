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
spring.jpa.database=MYSQL

spring.datasource.url=jdbc:mysql://mysql:3306/demo?useSSL=false
spring.datasource.username=root
spring.datasource.password=p4SSW0rd
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

* Dockerfile example
```
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
```
mvn clean install docker:build
```

* Docker Run MySQL Container
```
docker run -d \                                                                       
    --name demo-mysql \
    -e MYSQL_ROOT_PASSWORD=p4SSW0rd \
    -e MYSQL_DATABASE=demo \
    -p 3306:3306 mysql:latest
```

* Docker Run App Container
```
docker run -d \
    --name app-container \
    --link demo-mysql:mysql
    -p 8080:8080
    -p 8000:8000
    demo-app
```

* Docker Run MySQL Container with persistent data
```
docker run -d -p 3306:3306 -v /path/in/host:/var/lib/mysql mysql:latest
```

* Docker-compose file
```
version: '2'
services:
    demoapp-app:
        image: demoapp
        external_links:
            - demoapp-mysql:mysql
        ports:
            - 8080:8080
            - 8000:8000
    demoapp-mysql:
        image: mysql
        environment:
            - MYSQL_HOST=localhost
            - MYSQL_ROOT_PASSWORD=p4SSW0rd
            - MYSQL_DATABASE=demo
        ports:
            - 3306:3306
```
* An other example
```
    demoapp-app:
        image: demo-app
        links:
            - demoapp-mysql
        external_links:
            - demoapp-mysql:mysql
        ports:
            - 8080:8080
            - 8000:8000
    demoapp-mysql:
        image: mysql
        environment:
          MYSQL_ROOT_PASSWORD: 'p4SSW0rd'
          MYSQL_USER: dev
          MYSQL_PASSWORD: 'p4SSW0rd'
          MYSQL_DATABASE: demo
        ports:
            - 3306:3306
```

### Deboguer une application déployée dans un conteneur Docker
En se basant sur [ce chapitre de la doc officielle](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-running-your-application.html):
Pour démarrer l'application en mode Debug; il suffit d'ajouter ces paramètres dans la ligne d'exécution:
```
-Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n
```

Donc le nouveau Dockerfile sera:
```
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

