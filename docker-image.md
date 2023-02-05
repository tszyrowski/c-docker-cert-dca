# Docker Image
**`Docker image`** is a file with code and components needed to run software in a container.
- `layered file system`- each layer containing only differences from the previous layer
- `read only layers` - one or more, while `container` adds one additional writable layer.
- `sharing the same layers` allowed by layered file system
  - smaller overall storage
  - faster image transfer
  - faster image build

`$ docker image pull nginx `
```
Using default tag: latest
latest: Pulling from library/nginx
01b5b2efb836: Pull complete 
db354f722736: Pull complete 
abb02e674be5: Pull complete 
214be53c3027: Pull complete 
a69afcef752d: Pull complete 
625184acb94e: Pull complete 
Digest: sha256:c54fb26749e49dc2df77c6155e8b5f0f78b781b7f0eadd96ecfabdcdfa5b1ec4
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```
`$ docker image history nginx` - list layers used to build image
```
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
9eee96112def   33 hours ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      33 hours ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B        
<missing>      33 hours ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
<missing>      33 hours ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B        
<missing>      33 hours ago   /bin/sh -c #(nop) COPY file:e57eef017a414ca7…   4.62kB    
<missing>      33 hours ago   /bin/sh -c #(nop) COPY file:abbcbf84dc17ee44…   1.27kB    
<missing>      33 hours ago   /bin/sh -c #(nop) COPY file:5c18272734349488…   2.12kB    
<missing>      33 hours ago   /bin/sh -c #(nop) COPY file:7b307b62e82255f0…   1.62kB    
<missing>      33 hours ago   /bin/sh -c set -x     && addgroup --system -…   61.3MB    
<missing>      33 hours ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bullseye   0B        
<missing>      33 hours ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.9        0B        
<missing>      33 hours ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.23.3     0B        
<missing>      33 hours ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B        
<missing>      36 hours ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      36 hours ago   /bin/sh -c #(nop) ADD file:1d256392bb7afe694…   80.5MB 
```
## Dockerfile

`FROM` should be first 
`ENV` environments
`RUN` create a layer.
> Note if splitting `apt-get update` and following installs into multiple layers, the `apt-get update` should be done each time as the previous layer maybe re-used hence commands will not run.
`CMD` is default but optional

example
```Dockerfile
FROM ubuntu:bionic

ENV NGINX_VERSION 1.14.0-0ubuntu1.11

RUN apt-get update && apt-get install -y curl
RUN apt-get update && apt-get install -y nginx=$NGINX_VERSION

CMD ["niginx", "-g", "daemon off;"]
```
to run it: `docker build -t custom-nginx .` where `-t` is a tag and final `.` is a directory where to find it