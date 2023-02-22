FROM amazoncorretto:11-alpine3.17 as builder
WORKDIR /tmp/app
COPY ../app/ .
RUN ./gradlew assemble

FROM amazoncorretto:11-alpine3.17 as runtime
WORKDIR /opt/amazoncorretto/app
COPY --from=builder tmp/app/build/libs/rest-service-0.0.1-SNAPSHOT.jar .
CMD ["java", "-jar", "./rest-service-0.0.1-SNAPSHOT.jar"]
LABEL org.opencontainers.image.source https://github.com/${GIT_PATH:-ericheresi/hello-springrest}