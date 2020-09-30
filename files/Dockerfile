FROM openjdk:8

RUN apt-get update && apt-get -y install git
RUN cd /tmp \
    && git clone https://github.com/buzypi/spring-jpa \
    && cd spring-jpa \
    && ./mvnw clean install \
    && mkdir -p /srv \
    && mv target/*.jar /srv

WORKDIR /srv

CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]