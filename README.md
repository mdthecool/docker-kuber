# docker-kuber


Apk update 
Apk add apache2 


cat Dockerfile 
FROM alpine
RUN apk update
RUN apk add apache2
 

Build docker image : 

### docker image build -t app:latest  -t app:1 .

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

### . represents the docker image build context where Dockerfile is present. 

### The direcotry location is treated as docker context. Dockerfile should be with in the context. 

### dicker image prune   - Remove all dangling images 

#### docker image build --no-cache to make sure clean building of image without cache usage

#### Things are related and to optimze RUN command can be changed using ; or && 


## Docker process management 

#### Running docker httpd in FOREGROUND so that docker can monitor httpd process
docker container run -d app httpd -D FOREGROUND 

process pid = 1 is init process /sbin/init noembd norestore 

But, in the docker httpd is root process. The process run on host system, the process isolation is not practial, but docker presents its view of the process. 

Docker client talks to docker using socker file created /var/docker 

docker-container-shim is the process that represents the docker container in the host machine. 

$ ps -aef | grep shim 
root      1885  2431  0 06:23 ?        00:00:00 docker-containerd-shim 3680668f4fbe52426d9b7bdc700caaa6ec43ed6896da4e0e11f0d69904ad920a /var/run/docker/libcontainerd/3680668f4fbe52426d9b7bdc700caaa6ec43ed6896da4e0e11f0d69904ad920a docker-runc 

docker-containerd  is the parent process to docker containers. 


root      2431  2392  0 01:26 ?        00:00:16 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc

dockerd is the parent process for docker-containerd . 

#### dockerd --> docker-containerd --> docker-container-shim 

Sduo process isolation because of the above structure. 


##  Starting main process from the Docker file 

CMD ["httpd", "-D", "FOREGROUND"] 

Why we use it as array instead of httpd -D "FOREGROUND" it will be /bin/sh -c  httpd -D "FOREGROUND" which will be a shell command that gets exeucted ????

openssl docker image, please check it might be useful. 

## Managing configuration of service : 

### Environment varialble 

-e APACHE_PORT=8888 can be passed from outside when you run container. Also make sure you have scripts to handle that. 

docker run -d  -e APACHE_PORT=8888 -p 9001:8888 app  

docker image pull ghost:1-alpine

docker image pull mysql:5.7 

#### MYSQL_ROOT_PASSWORD  MYSQL_ALLOW_ENTRY_PASSWORD MYSQL_RANDOM_ROOT_PASSWORD  are the env that are required for mysql. 

docker container run -d -e MYSQL_ROOT_PASSWORD=welcome1 mysql:5.7  

Looking at container logs : 

docker container logs -f 09bdf43c0784  



### Volumes and mount management  

 This can be used for externalize the configuraiton or importent files ectc, 

Mounts will represents volumes on the docker container. 

/var/log/apache2  can be mounted home. 

"Mounts": [
            {
                "Type": "volume",
                "Name": "d09e682f4babe908ac445fcf6b37c85bd58fd3b64defc140edaec7fc6993e53f",
                "Source": "/var/lib/docker/volumes/d09e682f4babe908ac445fcf6b37c85bd58fd3b64defc140edaec7fc6993e53f/_data",
                "Destination": "/var/log/apache2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ]
        

#### 1 docker run -d -v /var/log/apache2 app
 
Below is the default mount location: 

"/var/lib/docker/volumes/d09e682f4babe908ac445fcf6b37c85bd58fd3b64defc140edaec7fc6993e53f/_data", 
       
#### 2. mysql image by default defines volume for the DB data. 
VOLUME /var/lib/mysql in Docker file 

Both above ones are good for backup but not for resume of the data. 

#### 3. specify the namespace for the volume : example customerdb   (NAMED VOLUMES)

docker container run -d -e MYSQL_ROOT_PASSWORD=welcome1  -v customerdb:/var/lib/mysql  mysql:5.7 

"Mounts": [
            {
                "Type": "volume",
                "Name": "customerdb",
                "Source": "/var/lib/docker/volumes/customerdb/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ]
        
#### "Source": "/var/lib/docker/volumes/customerdb/_data", known location to "Destination": "/var/lib/mysql",

Starting mysql prompt from using docker : 

docker exec -it c9e80dcdafa6 mysql -uroot -pwelcome1 




