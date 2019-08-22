# bundle-java-and-go-on-docker
Hi, I have written this code i hope it will perhaps guide you on the foundation of bundling java and go code to docker and using Nginx as lond balancer to distribute traffic across both app.
### My docker-compose file ####
cat docker-compose.yml
version: '3.4'

services:
  nginx:
   container_name: trivago-nginx
   image: nginx:alpine
   restart: always
   ports:
        - "80:80"
   volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf

   links:
        - java-webserver
        - golang-webserver

   depends_on:
        - java-webserver
        - golang-webserver

  java-webserver:
    build:
      context: .
      dockerfile: java-Dockerfile
    working_dir: /home/ubuntu/trivago/target
    image: java-webserver
    hostname: java-webserver
    ports:
        - 8080
  golang-webserver:
    build:
      context: .
      dockerfile: go-Dockerfile
    working_dir: /home/ubuntu/trivago/
    image: ggolang-webserver
    hostname: golang-webserver
    ports:
        - 8080
  ########## My Golan Docker file ###
  FROM golang:1.12.0-alpine3.9
MAINTAINER Lloyd
COPY . golang-webserver /home/ubuntu/trivago/
COPY . hosts /
ENV HOSTNAME  golang-webserver
EXPOSE 8080

CMD ["/home/ubuntu/trivago/golang-webserver"]

####### My Java-Dockerfile ###
FROM openjdk:12-alpine

MAINTAINER Lloyd
RUN addgroup -g 1000 -S www-data \
 && adduser -u 1000 -D -S -G www-data www-data
COPY target/java-webserver.jar /home/ubuntu/trivago/target/java-webserver.jar
COPY . hosts /
ENV HOSTNAME  java-webserver
EXPOSE 8080

CMD ["java", "-jar", "/home/ubuntu/trivago/target/java-webserver.jar"]

###### My host file for both app ####
127.0.0.1   localhost
172.17.0.7  golang-webserver
172.17.0.8  java-webserver

####### My Nginx Config file ####

worker_processes 1;

events { worker_connections 1024; }
http {
    upstream backend {
        server java-webserver weight=3;
        server golang-webserver;
    }

    server {
        listen 80;
        location /java-webserver {
          proxy_pass http://java-webserver:8080/;

        }
        location /golang-webserver {
          proxy_pass http://golang-webserver:8080;
        }
    }
}
