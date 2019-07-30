# docker-kuber


Apk update 
Apk add apache2 


cat Dockerfile 
FROM alpine
RUN apk update
RUN apk add apache2
 

Build docker image : 

docker image build -t app:latest  -t app:1 .

docker images
REPOSITORY                                    TAG                 IMAGE ID            CREATED             SIZE
app                                           1                   a332e4655d9f        7 seconds ago       10.6MB
app                                           latest              a332e4655d9f        7 seconds ago       10.6MB
mdthecool/nginx                               latest              3a6092de8893        20 hours ago        145MB


By default this image is saved in /var/lib/docker. 

This can be pushed to docker hub by explict command. 

Above command creates two tags :latest :1. 

Alpine is production grade OS with less image size. 


DOCKER COMMAND:  

$ docker image ls app 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
app                 2                   b37655234215        38 seconds ago      10.6MB
app                 latest              b37655234215        38 seconds ago      10.6MB
app                 1                   a332e4655d9f        23 minutes ago      10.6MB
$ 

