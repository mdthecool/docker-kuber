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
```
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
```
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
```
$ ps -aef | grep shim 
root      1885  2431  0 06:23 ?        00:00:00 docker-containerd-shim 3680668f4fbe52426d9b7bdc700caaa6ec43ed6896da4e0e11f0d69904ad920a /var/run/docker/libcontainerd/3680668f4fbe52426d9b7bdc700caaa6ec43ed6896da4e0e11f0d69904ad920a docker-runc 
```
docker-containerd  is the parent process to docker containers. 

```
root      2431  2392  0 01:26 ?        00:00:16 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
```
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

```
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

```

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
```
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
```
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
```
smd-mac:ora-jul29-dock-kube smd$ kubectl scale rc helloworld-controller --replicas  2 
replicationcontroller "helloworld-controller" scaled
smd-mac:ora-jul29-dock-kube smd$ 

smd-mac:ora-jul29-dock-kube smd$ kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-btgrm   1/1       Running   0          14m
helloworld-controller-hp59c   1/1       Running   0          14m
smd-mac:ora-jul29-dock-kube smd$ 
```

No way to drain out and scale down, window or low traffic time is the onlyway .


Using configuration file also we can scale up: 

relica:3 

```
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
````

```
##### smd-mac:ora-jul29-dock-kube smd$  kubectl apply -f k8s/rc/rc-helloworld.yml 
replicationcontroller "helloworld-controller" configured
smd-mac:ora-jul29-dock-kube smd$ kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-btgrm   1/1       Running   0          20m
helloworld-controller-d4r9q   1/1       Running   0          34s
helloworld-controller-hp59c   1/1       Running   0          20m
smd-mac:ora-jul29-dock-kube smd$ 

```

#### app: helloworld will define the name of the lable for the POD. Even if you run as POD or RC.  

kubctl delete rc helloworld-controller 
will delete with the containers created part of POD or RC. 

Naming convention and unique names are required in K8. 

##### WINDOWS linux terminal . cmder 


### Deployment 

pod1 app:1
pod2 app:1
pod3 app:1


Rolling upgrades: 


pod1 app:1 --X
pod2 app:1
pod3 app:1
pod4 app:2 . -- > 

Undo of upgrades --> 


#### Replication Set: 

Deployment --> RC1 / RS1  

We can define the thresholds while doing upgrades dep1 max down :1 

While upgrading 

RC1 --> RC2 ( Replication Set ). 



Most common way of deploying services in K8 is deployments 

- Rolling upgrade 
- Downgrade 
- Thresholds while upgrading 
-


spec defines how each componenet of deployment should behave. 


```


smd-mac:ora-jul29-dock-kube smd$ kubectl apply -f k8s/dep/dep-helloworld.yml 
deployment.extensions "helloworld-dep" created
smd-mac:ora-jul29-dock-kube smd$ kubectl get pods
NAME                              READY     STATUS    RESTARTS   AGE
helloworld-dep-6b5cd64d8d-kggv8   1/1       Running   0          <invalid>
smd-mac:ora-jul29-dock-kube smd$ kubectl describe deployment helloworld 
Name:                   helloworld-dep
Namespace:              default
CreationTimestamp:      Thu, 01 Aug 2019 10:03:04 +0530
Labels:                 app=helloworld
Annotations:            deployment.kubernetes.io/revision=1
                        kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"extensions/v1beta1","kind":"Deployment","metadata":{"annotations":{},"name":"helloworld-dep","namespace":"default"},"spec":{"replicas":1...
Selector:               app=helloworld
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=helloworld
  Containers:
   web:
    Image:        nginx:mainline
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   helloworld-dep-6b5cd64d8d (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  10s   deployment-controller  Scaled up replica set helloworld-dep-6b5cd64d8d to 1

smd-mac:ora-jul29-dock-kube smd$ 


```

#### deployment creates Relica set not Replication controller 

```
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-6b5cd64d8d   1         1         1         1m
smd-mac:ora-jul29-dock-kube smd$ 

```

##### Scaling command of Deployemnt:

```

smd-mac:ora-jul29-dock-kube smd$ kubectl scale deploy helloworld-dep --replicas 10
deployment.extensions "helloworld-dep" scaled
smd-mac:ora-jul29-dock-kube smd$ kubectl get pods
NAME                              READY     STATUS              RESTARTS   AGE
helloworld-dep-6b5cd64d8d-5jf42   0/1       ContainerCreating   0          10s
helloworld-dep-6b5cd64d8d-7dxdf   1/1       Running             0          10s
helloworld-dep-6b5cd64d8d-kgbhd   1/1       Running             0          10s
helloworld-dep-6b5cd64d8d-kggv8   1/1       Running             0          7m
helloworld-dep-6b5cd64d8d-n7bbs   1/1       Running             0          10s
helloworld-dep-6b5cd64d8d-nr25b   1/1       Running             0          10s
helloworld-dep-6b5cd64d8d-svs8x   1/1       Running             0          10s
helloworld-dep-6b5cd64d8d-w2cdz   1/1       Running             0          10s
helloworld-dep-6b5cd64d8d-w9p6x   0/1       ContainerCreating   0          10s
helloworld-dep-6b5cd64d8d-wxjk6   1/1       Running             0          10s
smd-mac:ora-jul29-dock-kube smd$ 

```

#### k8 maintains history of the deployment 

```
smd-mac:ora-jul29-dock-kube smd$ kubectl rollout history deploy helloworld-dep 
deployments "helloworld-dep"
REVISION  CHANGE-CAUSE
1         <none>

smd-mac:ora-jul29-dock-kube smd$ 

smd-mac:ora-jul29-dock-kube smd$ kubectl rollout history deploy helloworld-dep --revision 1 
deployments "helloworld-dep" with revision #1
Pod Template:
  Labels:	app=helloworld
	pod-template-hash=2617820848
  Containers:
   web:
    Image:	nginx:mainline
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

smd-mac:ora-jul29-dock-kube smd$ 

```

We can do the same using deploy configuration 

```

spec:
  revisionHistoryLimit: 10
  replicas: 10

```
#### Rolling upgrade of the do deployments with new image - for the image 

``` 
kubectl set image deploy helloworld-dep web=nginx:alpine

smd-mac:ora-jul29-dock-kube smd$ kubectl set image deploy helloworld-dep web=nginx:alpine 
deployment.apps "helloworld-dep" image updated
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   5         5         3         8s
helloworld-dep-6b5cd64d8d   6         6         6         27m
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   6         6         4         10s
helloworld-dep-6b5cd64d8d   5         5         5         27m
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   6         6         4         11s
helloworld-dep-6b5cd64d8d   5         5         5         27m
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   7         7         5         13s
helloworld-dep-6b5cd64d8d   4         4         4         27m
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   8         8         6         15s
helloworld-dep-6b5cd64d8d   3         3         3         27m
smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   10        10        8         25s
helloworld-dep-6b5cd64d8d   1         1         1         27m
smd-mac:ora-jul29-dock-kube smd$ 



``` 


We can undo  image to mainline nginx image can also be done nby executiong the command 


```

smd-mac:ora-jul29-dock-kube smd$ kubectl rollout status deploy helloworld-dep 
deployment "helloworld-dep" successfully rolled out
smd-mac:ora-jul29-dock-kube smd$

smd-mac:ora-jul29-dock-kube smd$ kubectl rollout history deploy helloworld-dep --revision 1
deployments "helloworld-dep" with revision #1
Pod Template:
  Labels:	app=helloworld
	pod-template-hash=2617820848
  Containers:
   web:
    Image:	nginx:mainline
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>



smd-mac:ora-jul29-dock-kube smd$ kubectl rollout undo deploy helloworld-dep --to-revision 1 
deployment.apps "helloworld-dep" 
smd-mac:ora-jul29-dock-kube smd$ kubectl rollout status deploy helloworld-dep 
Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 9 of 10 updated replicas are available...
deployment "helloworld-dep" successfully rolled out
smd-mac:ora-jul29-dock-kube smd$ 

smd-mac:ora-jul29-dock-kube smd$ kubectl get rs 
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   0         0         0         13m
helloworld-dep-6b5cd64d8d   10        10        10        40m
smd-mac:ora-jul29-dock-kube smd$ 


``` 

##### You can do the rollout of new image using deployment yaml file as well. 

```

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: helloworld-dep
spec:
  revisionHistoryLimit: 10
  replicas: 1
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: web
        image: nginx:1
        imagePullPolicy: IfNotPresent
        ports:
        - name: nginxport
          containerPort: 80
````

Modified the image to nginx:1 instead of mainline or alpine. 

```
smd-mac:ora-jul29-dock-kube smd$ kubectl apply  -f k8s/dep/dep-helloworld.yml 

smd-mac:ora-jul29-dock-kube smd$ kubectl get rs
NAME                        DESIRED   CURRENT   READY     AGE
helloworld-dep-5cdb68cc76   0         0         0         23m
helloworld-dep-6b5cd64d8d   1         1         1         50m
smd-mac:ora-jul29-dock-kube smd$ 

``` 

# K8 Services 

*Services can enable communication with POD*

* Node port is port of the POD specified by the selector -- Selector is what you define in deployment as selection name - app:helloworld **

```
smd-mac:ora-jul29-dock-kube smd$ kubectl apply -f k8s/service/svc-helloworld.yml 
service "hw-svc" created

smd-mac:ora-jul29-dock-kube smd$ kubectl describe service hw-svc 
Name:                     hw-svc
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"hw-svc","namespace":"default"},"spec":{"ports":[{"nodePort":31001,"port":8888,...
Selector:                 app=helloworld
Type:                     NodePort
IP:                       10.104.58.204
Port:                     <unset>  8888/TCP
TargetPort:               nginxport/TCP
NodePort:                 <unset>  31001/TCP
Endpoints:                172.17.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
smd-mac:ora-jul29-dock-kube smd$ 

``` 

minicubeip:31001 will serve the nginx 

#### SCALE AND DEPLOY #### 

``` 

smd-mac:ora-jul29-dock-kube smd$ kubectl scale deploy helloworld-dep --replicas 2
deployment.extensions "helloworld-dep" scaled
smd-mac:ora-jul29-dock-kube smd$ kubectl get pods 
NAME                              READY     STATUS    RESTARTS   AGE
helloworld-dep-6b5cd64d8d-4j2s4   1/1       Running   0          1h
helloworld-dep-6b5cd64d8d-hpsjj   1/1       Running   0          21s
smd-mac:ora-jul29-dock-kube smd$ 


smd-mac:ora-jul29-dock-kube smd$ kubectl describe service hw-svc 
Name:                     hw-svc
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"hw-svc","namespace":"default"},"spec":{"ports":[{"nodePort":31001,"port":8888,...
Selector:                 app=helloworld
Type:                     NodePort
IP:                       10.104.58.204
Port:                     <unset>  8888/TCP
TargetPort:               nginxport/TCP
NodePort:                 <unset>  31001/TCP
Endpoints:                172.17.0.4:80,172.17.0.5:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
smd-mac:ora-jul29-dock-kube smd$ 
``` 

SERVICE IP .:  service PORT : 


IP:                       10.104.58.204 .  - This IP address is valid with in the cluster (minicube it should work )
Port:                     <unset>  8888/TCP
	
NODE PORT : NodePort:                 <unset>  31001/TCP 
	


* Looking at the logs of the POD 
```

smd-mac:ora-jul29-dock-kube smd$ kubectl logs -f helloworld-dep-6b5cd64d8d-4j2s4  
172.17.0.1 - - [01/Aug/2019:06:10:22 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36" "-"
2019/08/01 06:10:22 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.101:31001", referrer: "http://192.168.99.101:31001/"
172.17.0.1 - - [01/Aug/2019:06:10:22 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://192.168.99.101:31001/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36" "-"

```



smd-mac:ora-jul29-dock-kube smd$ kubectl logs -f helloworld-dep-6b5cd64d8d-4j2s4

##### if you hit log of requests using following command: 

smd-mac:ora-jul29-dock-kube smd$ while true; do curl http://192.168.99.101:31001/; done  

You can see logs flowing in both the POD's. 

#### Connecting to container 

smd-mac:ora-jul29-dock-kube smd$ kubectl exec -it helloworld-dep-6b5cd64d8d-4j2s4  -- sh 

If POD has multiple containers then --container web can be given. If not it will connect first container.

``` 
cat /etc/resolve.conf 
nameserver 10.96.0.10 
``` 

#### This named service is added to all so PODs can ping the services using name of the service "hw-svc"

** This is service discovery feature ** 



##### Controllor service 

##### Scheculer 

##### ECTD - DB



#####  [[ KUBE-PROXY   --> canal (mesh network components) ]]--> CREATES A SUBNET FOR A POD - 172.17.0.0 / 172.18.0.0 /172.18.0.0  -- to make sure there are no clash of IP Addresses 

##### MESHNETWORK .  ---> calico , flannel(oracle), cloud weave(google), overlay (Docker-swarm) - Networking solution between the K8 Nodes  - Clustering 



#### INFLIGHT SNAGGING #### 





![alt text](https://github.com/mdthecool/docker-kuber/blob/master/app/i1.png)



# NETWORKING DIAGRAM 


![alt text](https://github.com/mdthecool/docker-kuber/blob/master/app/i2.png)


Starting kube  setup.txt --- Follow the commands. 

root@k8smaster:/home/vagrant# kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.167.10.70  

```


root@k8smaster:/home/vagrant# kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.167.10.70 
[init] Using Kubernetes version: v1.15.1
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8smaster localhost] and IPs [192.167.10.70 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8smaster localhost] and IPs [192.167.10.70 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8smaster kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.167.10.70]
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 21.515185 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.15" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8smaster as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8smaster as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: wbz3xt.w48ssq9o5w00foep
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.167.10.70:6443 --token wbz3xt.w48ssq9o5w00foep \
    --discovery-token-ca-cert-hash sha256:4023e8b336f48a65a543c075d37cef6efadb29d60282d19760b2fad865f42889 
    
    
    
 ```
 
#### kubeadm join command  - This shoild be shared with people who needs to join the cluster - Nodes that needs to join this cluster. 
 
 ```
 
 kubeadm join 192.167.10.70:6443 --token wbz3xt.w48ssq9o5w00foep \
    --discovery-token-ca-cert-hash sha256:4023e8b336f48a65a543c075d37cef6efadb29d60282d19760b2fad865f42889 
    
    
    root@k8snode1:/home/vagrant#  kubeadm join 192.167.10.70:6443 --token wbz3xt.w48ssq9o5w00foep \
>     --discovery-token-ca-cert-hash sha256:4023e8b336f48a65a543c075d37cef6efadb29d60282d19760b2fad865f42889 
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.15" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

root@k8snode1:/home/vagrant# 

``` 



Canal is third party software should be conifgured manually as a POD. 



eth1 in canal.yaml is the only change we have to do. 

```

root@k8smaster:/vagrant# kubectl apply -f rbac.yml 
clusterrole.rbac.authorization.k8s.io/calico created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/canal-flannel created
clusterrolebinding.rbac.authorization.k8s.io/canal-calico created
root@k8smaster:/vagrant# kubectl apply -f canal.yml 
configmap/canal-config created
daemonset.extensions/canal created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
serviceaccount/canal created
root@k8smaster:/vagrant# 



```


#### Setup end to end jenkins with docker on master node:


```

➜  ora-jul29-dock-kube git:(master) ✗ cat build-machine.txt 

-------------------------------------------------Dockerfile--------------------------------------------------
FROM jenkins/jenkins:lts
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" \
    && apt-get update  -qq \
    && apt-get install docker-ce=17.09.0~ce-0~debian -y
RUN usermod -aG docker jenkins
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && mv kubectl /bin && chmod +x /bin/kubectl
RUN apt-get install -y git
----------------------------------------------------------------------------------------------------------------

 docker container run -d -v /var/run/docker.sock:/var/run/docker.sock -v /etc/kubernetes/admin.conf:/etc/config -e KUBECONFIG=/etc/config -v /vagrant/jenkins_home5:/var/jenkins_home -p 9090:8080 build-machine
➜  ora-jul29-dock-kube git:(master) ✗ 

``` 

#### -v /var/run/docker.sock:/var/run/docker.sock -- This makes sure that docker container in master talks to external dockerd that is running on the master node - Jenkins docker can not run dockerd. docker is just the client. 

```

root@k8smaster:/vagrant# docker exec f349b61d829cb35e5a3618103283c903d195a4d64d56041e0b5cbb5a74dc224c docker container ls 
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                               NAMES
f349b61d829c        build-machine            "/sbin/tini -- /us..."   4 minutes ago       Up 4 minutes        50000/tcp, 0.0.0.0:9090->8080/tcp   zealous_hawking
4dd4cc758df7        quay.io/coreos/flannel   "/opt/bin/flanneld..."   About an hour ago   Up About an hour                                        k8s_kube-flannel_canal-j6z8t_kube-system_d10e10de-b479-4e77-98bc-74304409fc5c_0
73c8f27da41b        eb516548c180             "/coredns -conf /e..."   About an hour ago   Up About an hour                                        k8s_coredns_coredns-5c98db65d4-pbkrx_kube-system_0bf3e113-faad-432c-a702-7d69b18baecd_0
899d8187881e        eb516548c180             "/coredns -conf /e..."   About an hour ago   Up About an hour                                        k8s_coredns_coredns-5c98db65d4-cnm4b_kube-system_3f378a24-82f9-4d22-b0fb-8757411fa4bd_0
3d6014ddee2f        k8s.gcr.io/pause:3.1     "/pause"                 About an hour ago   Up About an hour                                        k8s_POD_coredns-5c98db65d4-cnm4b_kube-system_3f378a24-82f9-4d22-b0fb-8757411fa4bd_0
3a1492d152ac        k8s.gcr.io/pause:3.1     "/pause"                 About an hour ago   Up About an hour                                        k8s_POD_coredns-5c98db65d4-pbkrx_kube-system_0bf3e113-faad-432c-a702-7d69b18baecd_0
3c19d1964c07        quay.io/calico/cni       "/install-cni.sh"        About an hour ago   Up About an hour                                        k8s_install-cni_canal-j6z8t_kube-system_d10e10de-b479-4e77-98bc-74304409fc5c_0
fe000d099e85        quay.io/calico/node      "start_runit"            About an hour ago   Up About an hour                                        k8s_calico-node_canal-j6z8t_kube-system_d10e10de-b479-4e77-98bc-74304409fc5c_0
2e8ecdedd8ca        k8s.gcr.io/pause:3.1     "/pause"                 About an hour ago   Up About an hour                                        k8s_POD_canal-j6z8t_kube-system_d10e10de-b479-4e77-98bc-74304409fc5c_0
7f2fffa30e26        89a062da739d             "/usr/local/bin/ku..."   2 hours ago         Up 2 hours                                              k8s_kube-proxy_kube-proxy-z6j6x_kube-system_d1e71d88-7785-4fa5-aee8-a16b08065a80_0
15794cab57f3        k8s.gcr.io/pause:3.1     "/pause"                 2 hours ago         Up 2 hours                                              k8s_POD_kube-proxy-z6j6x_kube-system_d1e71d88-7785-4fa5-aee8-a16b08065a80_0
df98af99abde        b0b3c4c404da             "kube-scheduler --..."   2 hours ago         Up 2 hours                                              k8s_kube-scheduler_kube-scheduler-k8smaster_kube-system_ecae9d12d3610192347be3d1aa5aa552_0
34cc4ef6ff24        2c4adeb21b4f             "etcd --advertise-..."   2 hours ago         Up 2 hours                                              k8s_etcd_etcd-k8smaster_kube-system_e5cd44264957d2c95fa822f9036af82e_0
f45b0d38bf1f        d75082f1d121             "kube-controller-m..."   2 hours ago         Up 2 hours                                              k8s_kube-controller-manager_kube-controller-manager-k8smaster_kube-system_bbfba61185a8e7737ec27dbd6735a1d8_0
8bb6a765c0e5        68c3eb07bfc3             "kube-apiserver --..."   2 hours ago         Up 2 hours                                              k8s_kube-apiserver_kube-apiserver-k8smaster_kube-system_3acb31820e6af7646ebbd616e5b8a20f_0
f69590ce7eae        k8s.gcr.io/pause:3.1     "/pause"                 2 hours ago         Up 2 hours                                              k8s_POD_kube-scheduler-k8smaster_kube-system_ecae9d12d3610192347be3d1aa5aa552_0
eedd600a1561        k8s.gcr.io/pause:3.1     "/pause"                 2 hours ago         Up 2 hours                                              k8s_POD_etcd-k8smaster_kube-system_e5cd44264957d2c95fa822f9036af82e_0
c73268cbf9a5        k8s.gcr.io/pause:3.1     "/pause"                 2 hours ago         Up 2 hours                                              k8s_POD_kube-controller-manager-k8smaster_kube-system_bbfba61185a8e7737ec27dbd6735a1d8_0
598e63a3c1a1        k8s.gcr.io/pause:3.1     "/pause"                 2 hours ago         Up 2 hours                                              k8s_POD_kube-apiserver-k8smaster_kube-system_3acb31820e6af7646ebbd616e5b8a20f_0


```

#### . -v /etc/kubernetes/admin.conf:/etc/config -e KUBECONFIG=/etc/config   This is to make sure kubctl can connect to kube cluster that master of 

```
root@k8smaster:/vagrant# docker exec f349b61d829cb35e5a3618103283c903d195a4d64d56041e0b5cbb5a74dc224c kubectl get nodes 
NAME        STATUS   ROLES    AGE    VERSION
k8smaster   Ready    master   140m   v1.15.1
k8snode1    Ready    <none>   102m   v1.15.1
root@k8smaster:/vagrant# 


docker exec f349b61d829cb35e5a3618103283c903d195a4d64d56041e0b5cbb5a74dc224c cat /var/jenkins_home/secrets/initialAdminPassword 
579bd12812434ff2ab85fa6b69631181
root@k8smaster:/vagrant# 

```
#### https://github.com/AdityaSP/docker-devops 




Jenkins code is in Aditya github project. 


#### Swith context 

``` 
➜  ora-jul29-dock-kube git:(master) ✗ kubectl config get-contexts 
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
➜  ora-jul29-dock-kube git:(master) ✗ kubectl config use-context kubernetes-admin@kubernetes 
Switched to context "kubernetes-admin@kubernetes".
➜  ora-jul29-dock-kube git:(master) ✗ kubectl config get-contexts                            
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
➜  ora-jul29-dock-kube git:(master) ✗ 

```



# K8 Volumes 

##### Volumes (in the deoployment yaml) -->  Persistenet Volume Claim ( Logical config for pyhsical ) --- Persistent Volume 




sp.aditya@gmail.com








