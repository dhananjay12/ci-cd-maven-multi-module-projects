FROM java:8-jdk-alpine

COPY ./target/product-service*.jar /usr/app/product-service.jar

WORKDIR /usr/app

RUN sh -c 'touch product-service.jar'

ENTRYPOINT ["java","-jar","product-service.jar"]