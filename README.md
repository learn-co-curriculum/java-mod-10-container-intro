# A Brief Overview of Containers

## Learning Goals

- Understanding basics of Docker


## Introduction


Read through some brief overviews to get a better understanding of where Docker is situated in the software space.

- [Docker Pitch](https://www.docker.com/resources/what-container/)
- [Docker Overview](https://docs.docker.com/get-started/overview/)


## Docker Basics

The following few sections should comprise a basic Docker Primer of all the most commonly used Docker commands and
Dockerfile syntax.


### Docker usage

Common docker commands:

- `docker ps -a` – list all docker containers
- `docker run -p <destination_port:source_port> -e ENV=’env’ -d <image>` – start new container instance as daemon from image with port bindings and environmental variables
- `docker stop <ID/NAME>` – stop running container
- `docker start <ID/NAME>` – start stopped container
- `docker rm -f <ID/NAME>` – remove container(forcefully if running)
- `docker pull <image>` – download image from hosting registry
- `docker push <image>` – push image to registry(if access allowed)
- `docker tag <image:sha_hash> <image:tag_version>` – tag images based on previous tags or hashes
- `docker logs <ID/NAME>` – list STDOUT and STDERR for container
- `docker exec -it <ID/NAME> ​/bin/bash` – open an interactive bash session in a container
- `docker build -t <image:tag> .` – builds image from Dockerfile in current directory


### Dockerfile usage

Dockerfile is a declarative syntax for building docker images. There are other ways to build images if needed(Packer),
but Dockerfile is the standard.

Dockerfile should be treated as Configuration Management, and tracked in source control

Common Dockerfile commands:

- `FROM` – pulls base image
- `ENV` – passes environmental variables
- `RUN` – runs arbitrary commands
- `ADD/COPY` – copies local files into image; use copy; multi-stage builds
- `EXPOSE` – makes container ports available
- `CMD/ENTRYPOINT` – specifies binary to run

Dockerfile example:

``` text
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]
```

Dockerfile multi-stage example:

``` text
FROM openjdk:8-jdk-alpine as build
WORKDIR /workspace/app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src

RUN ./mvnw install -DskipTests
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=/workspace/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]
```
