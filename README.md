# Lab Creating Optimized Spring Boot application Docker Image

Since version 2.3 (from 2.4 all Jar files are layered by default) Spring frameworks started to support layered JAR files as optimization technique.
This can be used for optimizing containerized Spring Boot Applications.
---

## Traditional build with syntax:
```dockerfile
FROM openjdk:11-jdk-slim
EXPOSE 8080
ARG JAR_FILE=target/demo-app-1.0.0.jar
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
Process of building container can be optimized and size of the image can be trimmed by exploding **Fat Jar**.

Application start time can also improve if we build image from decompressed Jar file.

---
## Optimized approach

**Fat Jar** created originally by Spring Boot can impact startup time of the containerized application, this can be reduced by exploding content of the **JAR** file and creating layered Docker image.

Layering the container image provides improvement to the building process by leveraging the Docker caching mechanism.

After exploding the JAR file and constructing layered Spring Boot image, we can rebuild only the layer that changed instead of the whole image as in the traditional **Fat Jar** approach.

[Dockerfile](./Dockerfile) :
```dockerfile
# define the builder contaier layer
FROM openjdk:11-jdk-slim as builder
WORKDIR application
# copy packaged JAR file to the Builder container layer
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
# decompressing the JAR file into layers
RUN java -Djarmode=layertools -jar application.jar extract

# define the final image manifest
FROM openjdk:11-jre-slim
# copy uncompressed Spring Boot application to root of final image
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/application/ ./
# expose port application is listening on
EXPOSE 8080
# starting application
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```


---
### To read:
[Creating Docker images with Spring Boot 2.3.0.M1](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1)

[Creating Efficient Docker Images with Spring Boot 2.3](https://spring.io/blog/2020/08/14/creating-efficient-docker-images-with-spring-boot-2-3)

[Creating Docker Images with Spring Boot](https://www.baeldung.com/spring-boot-docker-images)