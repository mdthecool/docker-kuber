# docker-kuber


Docker Open container spec implementation of linux core cotainers which is very poplular. Cloudfoundry , Rocket (RHEL) . 

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

#### Source path to destination path  BIND 

docker run -d -v /tmp/logs /var/log/apache2 app

"Mounts": [
            {
                "Type": "bind",
                "Source": "/tmp/logs",
                "Destination": "/var/log/apache2",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        
        binds 
        
        
        
### Docker Networking 

Bridge , host and none - default network adaptors 

$ docker network ls 
NETWORK ID          NAME                DRIVER              SCOPE
278d706287e2        bridge              bridge              local
0999ce21d6ab        host                host                local
0559eb5c0b49        none                null                local
$  



docker network inspect bridge

$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
278d706287e2        bridge              bridge              local
0999ce21d6ab        host                host                local
0559eb5c0b49        none                null                local
095b8f49d407        w1-dn-net           bridge              local
$ 



$ docker network inspect w1-dn-net  
[
    {
        "Name": "w1-dn-net",
        "Id": "095b8f49d4077b8526f0fc4eb872dbec5a6b36aad1903e2300d6357f205faf10",
        "Created": "2019-07-31T04:36:45.007625227Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
$ 


### Remove w1 and db from default bridge network and move to newly created w1-dn-net. 

docker network connect w1-dn-net w1 


$ docker container inspect w1 | grep -i ipaddress 
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.4",
                    "IPAddress": "172.18.0.2",
$ 

"Gateway": "172.18.0.1"    -      "IPAddress": "172.18.0.2", 

Now gateway 172.18.0 - can connect to IPAddress 172.18.0.2



$ docker container inspect db | grep -i ipaddress 
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.6",
                    "IPAddress": "172.17.0.6",
$  


Moving DB docker from bridge to  w1-dn-net  

$ docker network disconnect bridge db 
$ docker container inspect db | grep -i ipaddress 
            "SecondaryIPAddresses": null,
            "IPAddress": "",
$ 

$ docker network connect w1-dn-net  db 
$ docker container inspect db | grep -i ipaddress 
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.3",
$ 


$ docker network create w2-db-net 
c1b8d7e9d858a31ccd7018199490b58646666fd11b94db6054dc1fab7ec739f8
$ docker network inspect w2-db-net  
[
    {
        "Name": "w2-db-net",
        "Id": "c1b8d7e9d858a31ccd7018199490b58646666fd11b94db6054dc1fab7ec739f8",
        "Created": "2019-07-31T04:46:25.792086809Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
$ 



$ docker network disconnect bridge w2
$ docker network connect w2-db-net w2
$ docker network connect w2-db-net db 
$ docker container inspect db | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.3",
                    "IPAddress": "172.19.0.3",
$ 



$ docker container inspect db | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.3",
                    "IPAddress": "172.19.0.3",
$ docker container inspect db | grep -i ipaddress 
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.3",
                    "IPAddress": "172.19.0.3",
$ docker container exec w2 pint -c2 172.19.0.3 
oci runtime error: exec failed: container_linux.go:265: starting container process caused "exec: \"pint\": executable file not found in $PATH"

$ docker container exec w2 ping -c2 172.19.0.3 
PING 172.19.0.3 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.109 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.075 ms

--- 172.19.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.075/0.092/0.109 ms
$ 



### DNS Name Resolution Automatically happens by the name of the container. Docker by default has embedded nameserver.  


$ docker container exec w2 ping -c2 db         
PING db (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.120 ms

--- db ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.095/0.120 ms
$  


## Docker Blog setup using networking 

1. network
docker network create blog-nw

1. db
docker container run -d --name blog-db --network blog-nw -v /tmp/blog:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=welcome mysql:5.7 

2. ghost
docker container run -d --name blog-ghost --network blog-nw -p 9090:2368 -e database__client=mysql -e database__connection__host=blog-db -e database__connection__port=3306 -e database__connection__user=root -e database__connection__password=welcome -e database__connection__database=ghost ghost:1-alpine





#### $ docker exec -it blog-db mysql -uroot -pwelcome
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 35
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show all;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'all' at line 1
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| ghost              |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> 




## Docker shortcomings 

1. N/w 
    1. ip clash 
    2. No node awareness 
 2. Better container managmement aacorss nodes (dockerd agent)
 
 3. Application scalaing 
 
 4. Load balacing 
 
 We need a tool manage multiple dockerds running on different VM / hosts 
 
 5. Better volume management independent of VM/ hosts -
 
 
 ##### Solutions: Docker-swarm : From docker opensource
 
 $ docker info
Containers: 15
 Running: 15
 Paused: 0
 Stopped: 0
Images: 25
Server Version: 17.09.0-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
#### Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 06b9cb35161009dcb7123345749fef02f7cea8e0
runc version: 3f2f8b84a77f73d38244dd690525642a72156c64
init version: N/A (expected: )
Security Options:
 seccomp
  Profile: default
Kernel Version: 4.9.64
Operating System: Buildroot 2017.11
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 1.953GiB
Name: minikube
ID: B2GG:OGQH:MSMT:QFQJ:RHD2:5U2G:VTKH:V5EB:CJKS:SRBX:VXBW:EO6E
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Username: mdthecool1
Registry: https://index.docker.io/v1/
Labels:
 provider=virtualbox
Experimental: false
Insecure Registries:
 10.96.0.0/12
 127.0.0.0/8
Live Restore Enabled: false

$ 

# KUBE 


YAML: - array space to represents data 




#### AGENDA

1. Single cluster node = terminologies of K8S 
2. communication 
3. Archi K8S 
4. Multinode cluster 
 

##### kubectl version 


smd-mac:.kube smd$ kubectl version 
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.11", GitCommit:"637c7e288581ee40ab4ca210618a89a555b6e7e9", GitTreeState:"clean", BuildDate:"2018-11-26T14:38:32Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"", Minor:"", GitVersion:"v1.9.0", GitCommit:"925c127ec6b946659ad0fd596fa959be43f0cc05", GitTreeState:"clean", BuildDate:"2018-01-26T19:04:38Z", GoVersion:"go1.9.1", Compiler:"gc", Platform:"linux/amd64"}
smd-mac:.kube smd$ 


kubectl commandline tool for the orchestration needs. 

#### k8 configureation is present .kube/config 

Context is single k8 master representation. 

#### smd-mac:.kube smd$ cat ~/.kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/smd/.minikube/ca.crt
    server: https://192.168.99.101:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    as-user-extra: {}
    client-certificate: /Users/smd/.minikube/client.crt
    client-key: /Users/smd/.minikube/client.key
smd-mac:.kube smd$ 

#### POD Creation 

smd-mac:ora-jul29-dock-kube smd$ kubectl apply -f k8s/pod/pod-helloworld.yml  
pod "my-nginx" created


#### smd-mac:ora-jul29-dock-kube smd$ kubectl describe pod my-nginx 
Name:         my-nginx
Namespace:    default
Node:         minikube/192.168.99.101
Start Time:   Wed, 31 Jul 2019 14:33:43 +0530
Labels:       app=helloworld
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"helloworld"},"name":"my-nginx","namespace":"default"},"spec":{"containers...
Status:       Running
IP:           172.17.0.4
Containers:
  web:
    Container ID:   docker://d41045a8158d2981bc07ba53ef221f5be42f18ca5d23c630b713a69f60f65b5a
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:482ead44b2203fa32b3390abdaf97cbdc8ad15c07fb03a3e68d7c35a19ad7595
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 31 Jul 2019 14:33:50 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-k7zt6 (ro)
Conditions:
  Type           Status
  Initialized    True 
  Ready          True 
  PodScheduled   True 
Volumes:
  default-token-k7zt6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-k7zt6
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              9m    default-scheduler  Successfully assigned my-nginx to minikube
  Normal  SuccessfulMountVolume  9m    kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-k7zt6"
  Normal  Pulling                9m    kubelet, minikube  pulling image "nginx:alpine"
  Normal  Pulled                 9m    kubelet, minikube  Successfully pulled image "nginx:alpine"
  Normal  Created                9m    kubelet, minikube  Created container
  Normal  Started                9m    kubelet, minikube  Started container
smd-mac:ora-jul29-dock-kube smd$ 


#### This reflects in the minikube - POD NAME is part of the docker container: 
$ docker ps -a
CONTAINER ID        IMAGE                                         COMMAND                  CREATED             STATUS                        PORTS               NAMES
d41045a8158d        nginx                                         "nginx -g 'daemon ..."   4 minutes ago       Up 4 minutes                                      k8s_web_my-nginx_default_188a8e9c-b372-11e9-809f-08002788c05b_0



name of the container is web , pod name is my-nginx 


#### The POD also a container is a demon for the POD which manages all the containers related to POD 

359c01ff2b49        gcr.io/google_containers/pause-amd64:3.0      "/pause"                 4 minutes ago       Up 4 minutes                                      k8s_POD_my-nginx_default_188a8e9c-b372-11e9-809f-08002788c05b_0


Docker container nginx does not get IP. 
## Networking prespective POD network is parent network for the all the containers of the POD (example NGINX, MYSQL etc.. ) useful for following reasons 

 - load balancing 
 - management of the communication with in POD management 


VETH - Virtual Ethernet Adaptors - Similar to that POD starts VIP

If POD container dies all the containers of POD will die. K8 will start the POD container as well as all needed containers as well. 

# Smallest management entity in K8 is POD

While doing horizantal scalaing should be done at POD level. 

pod is unit of scaling. 

###### delete POD 
smd-mac:ora-jul29-dock-kube smd$ kubectl delete pod my-nginx 
pod "my-nginx" deleted
smd-mac:ora-jul29-dock-kube smd$

#####  replicas: 2 - with template is way to replicate a pod


smd-mac:ora-jul29-dock-kube smd$  kubectl apply -f k8s/rc/rc-helloworld.yml 
replicationcontroller "helloworld-controller" created
smd-mac:ora-jul29-dock-kube smd$ 

Two containers spun 

$ docker ps -a
CONTAINER ID        IMAGE                                         COMMAND                  CREATED             STATUS                     PORTS               NAMES
8be129ccd7a9        nginx                                         "nginx -g 'daemon ..."   6 minutes ago       Up 6 minutes                                   k8s_nginx_helloworld-controller-hp59c_default_58dab4e4-b37e-11e9-809f-08002788c05b_0
8ee137a7d8cc        nginx                                         "nginx -g 'daemon ..."   6 minutes ago       Up 6 minutes                                   k8s_nginx_helloworld-controller-btgrm_default_58da3fa7-b37e-11e9-809f-08002788c05b_0


###### Replication of containers : 
smd-mac:ora-jul29-dock-kube smd$ cat   k8s/rc/rc-helloworld.yml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: helloworld-controller
spec:
  replicas: 2
  selector:
    app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - name: nginxport
          containerPort: 80

#### kubctl scale replicas is very simple --relicas 5 with rc. 


smd$ kubectl scale rc helloworld-controller --replicas  5 
replicationcontroller "helloworld-controller" scaled
smd-mac:ora-jul29-dock-kube smd$ kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-btgrm   1/1       Running   0          12m
helloworld-controller-f24jf   1/1       Running   0          39s
helloworld-controller-hp59c   1/1       Running   0          12m
helloworld-controller-hq5j5   1/1       Running   0          39s
helloworld-controller-l45jm   1/1       Running   0          39s
smd-mac:ora-jul29-dock-kube smd$ 




#### SCALE DOWN 

smd-mac:ora-jul29-dock-kube smd$ kubectl scale rc helloworld-controller --replicas  2 
replicationcontroller "helloworld-controller" scaled
smd-mac:ora-jul29-dock-kube smd$ 

smd-mac:ora-jul29-dock-kube smd$ kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-btgrm   1/1       Running   0          14m
helloworld-controller-hp59c   1/1       Running   0          14m
smd-mac:ora-jul29-dock-kube smd$ 


No way to drain out and scale down, window or low traffic time is the onlyway .


Using configuration file also we can scale up: 

relica:3 


apiVersion: v1
kind: ReplicationController
metadata:
  name: helloworld-controller
spec:
  replicas: 3
  selector:
    app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - name: nginxport
          containerPort: 80
          
          
##### smd-mac:ora-jul29-dock-kube smd$  kubectl apply -f k8s/rc/rc-helloworld.yml 
replicationcontroller "helloworld-controller" configured
smd-mac:ora-jul29-dock-kube smd$ kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-btgrm   1/1       Running   0          20m
helloworld-controller-d4r9q   1/1       Running   0          34s
helloworld-controller-hp59c   1/1       Running   0          20m
smd-mac:ora-jul29-dock-kube smd$ 


#### app: helloworld will define the name of the lable for the POD. Even if you run as POD or RC. 

kubctl delete rc helloworld-controller 
will delete with the containers created part of POD or RC. 

Naming convention and unique names are required in K8. 


