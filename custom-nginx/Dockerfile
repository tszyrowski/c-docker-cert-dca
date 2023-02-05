# simple ndingx
FROM ubuntu:bionic

ENV NGINX_VERSION 1.14.0-0ubuntu1.11

RUN apt-get update && apt-get install -y curl
RUN apt-get update && apt-get install -y nginx=$NGINX_VERSION

RUN nginx -v

CMD ["niginx", "-g", "daemon off;"]