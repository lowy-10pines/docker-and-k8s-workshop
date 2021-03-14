FROM openjdk:8-jre
EXPOSE 8080
COPY ./passwordapi.jar /usr/src/passwordapi/passwordapi.jar
WORKDIR /usr/src/passwordapi
CMD ["java", "-jar", "passwordapi.jar"]