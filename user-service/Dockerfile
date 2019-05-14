FROM java:8-jdk-alpine

COPY ./target/user-service*.jar /usr/app/user-service.jar

WORKDIR /usr/app

RUN sh -c 'touch product-service.jar'

ENTRYPOINT ["java","-jar","product-service.jar"]