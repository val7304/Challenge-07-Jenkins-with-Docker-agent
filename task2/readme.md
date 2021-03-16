

# Challenge 7 - Jenkins with Docker agents  


Date limite de remise du challenge: le 07 mars 2021 

---
## Task 2: Jenkins Distributed builds with Docker agents
Durée estimée : **Katacoda:** 15 à 20 minutes. **Mise en place de l'agent:** 2 Heures

L'objectif ici est de configurer des builds Jenkins distribués dans lesquels Docker joue le rôle d'agent.
 - Suivre et appliquer les instructions du Katacoda [Building Docker Images using Jenkins](https://www.katacoda.com/courses/cicd/build-docker-images-using-jenkins). Ce Katacoda illustre l'utilisation d'un agent Docker pour construire une image Docker. Ce Katacoda  est intéressant dans la mesure où illustre le processus global de mise en place d'un agent Docker, il montre surtout comment configurer le Plugin Docker (step 2 du Katacoda) et comment ajouter l'agent (step 3 du Katacoda) au master Jenkins. Toutefois, l'environnement de l'agent est deja pré-configuré par l'auteur.
 - Construire par vous même un environnement d'agent Docker pour Jenkins. Suivre le tutorial suivant, par Ashu Rana, sur Medium, intitulé : [Using Docker Containers as Jenkins Build Slaves](https://medium.com/xebia-engineering/using-docker-containers-as-jenkins-build-slaves-a0bb1c9190d). Les mêmes idées sont également développées par Bibin Wilson, dans son article [How to Setup Docker Containers as Build Slaves for Jenkins](https://devopscube.com/docker-containers-as-build-slaves-jenkins/). L'auteur fournit une [vidéo Youtube explicative](https://youtu.be/yb6DodK6mbg).
 -----
**Mise en place du challenge:** 

	/**
	 * HOW TO SETUP DOCKER CONTAINER AS BUILD SLAVE FOR JENKINS 
	 * Mise en place en suivant les différentes sources proposées ci-dessus 
	**/
Utilisation de 2 Dockerfile
 1. pour le container DockerHost basé sur un CentOs 
 2. pour le container Jenkins 

## DockerHost:


Build du Dockerfile se trouvant dans le dossier /CentOs-Docker-as-build-slave/ : 

    PS D:\challenges\centOs> docker build -t dockertest .  

Run image: 

    docker run --privileged --name dockertest -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 -d  dockertest

Jump to the container: 

    PS D:\challenges\centOs> docker exec -it 786 /bin/bash
Start docker and asked his status: 

    [root@786dac9969ee /]# systemctl start docker
    [root@786dac9969ee /]# systemctl status docker
    [root@786dac9969ee /]# systemctl enable docker
    Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

 Change docker.service ExecStart values:

    [root@786dac9969ee /]# vi /lib/systemd/system/docker.service

i to enter in "Insert Mode"
replace: `ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock`
by: `ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock`
ESC + :wq (SAVE AND QUIT)

Reload && restart system: 

    [root@786dac9969ee /]# systemctl daemon-reload
    
    [root@786dac9969ee /]# service docker restart //command don't work => return bash: service: command not found
    [root@786dac9969ee /]# systemctl restart docker #use this instead of 'service docker restart'

Verify data *ExecStart* in docker.service:

    [root@786dac9969ee /]# cat /lib/systemd/system/docker.service
    [Unit]
    //... 
    ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock  
    //..
Verify if docker is well:

    [root@786dac9969ee /]# docker version
    Client: Docker Engine - Community
     Version:           20.10.5
     API version:       1.41
     Go version:        go1.13.15
    //...
Verify if the port is open: 

    [root@786dac9969ee /]# ps faux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root       355  0.0  0.0  11832  3008 pts/0    Ss   14:46   0:00 /bin/bash
    root       414  0.0  0.0  51748  3644 pts/0    R+   15:02   0:00  \_ ps faux
    root         1  0.0  0.0  43168  4856 ?        Ss   14:23   0:00 /usr/sbin/init
    root        19  0.0  0.0  39092  5408 ?        Ss   14:23   0:00 /usr/lib/systemd/systemd-journald
    ...
    ...
    root        46  0.0  0.4 766500 54656 ?        Ssl  14:23   0:01 /usr/bin/containerd
    dbus        80  0.0  0.0  58108  4320 ?        Ss   14:23   0:00 /usr/bin/dbus-daemon --system --address=systroot       218  0.0  0.7 1053336 97616 ?       Ssl  14:36   0:00 /usr/bin/dockerd -H tcp://0.0.0.0:4243 -H un

Vérification de la connexion sur le port 22 de l'hôte docker:

    #first in localhost: 
    [root@786dac9969ee /]# curl http://localhost:4243
    /version
    {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"20.10.5","Details":{"ApiVersion":"1.41","Arch":"amd64","BuildTime":"2021-03-02T20:32:17.000000000+00:00","Experimental":"false","GitCommit":"363e9a8","GoVersion":"go1.13.15","KernelVersion":"4.19.128-microsoft-standard","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.4.3","Details":{"GitCommit":"269548fa27e0089a8b8278fc4fc781d7f65a939b"}},{"Name":"runc","Version":"1.0.0-rc92","Details":{"GitCommit":"ff819c7e9184c13b7c2607fe6c30ae19403a7aff"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"20.10.5","ApiVersion":"1.41","MinAPIVersion":"1.12","GitCommit":"363e9a8","GoVersion":"go1.13.15","Os":"linux","Arch":"amd64","KernelVersion":"4.19.128-microsoft-standard","BuildTime":"2021-03-02T20:32:17.000000000+00:00"}

Find public ip from DockerHost : 

    valer@DESKTOP-L2RC63V MINGW64 /d/challenges/challenge-test/centOs
    $ docker inspect dockertest
      "Networks": {
                    "bridge": {
                        "IPAMConfig": null,
                        "Links": null,
                        "Aliases": null,
                        "NetworkID": "0d5f5d52498000933c2fadf42a322779138b707b927ca734390e985f169085ce",
                        "Gateway": "172.17.0.1",        
    	=>=>=>      "IPAddress": "172.17.0.2",
		    	    "IPPrefixLen": 16,
                        "IPv6Gateway": "",
                        "GlobalIPv6Address": "",
                        "GlobalIPv6PrefixLen": 0,
                        "MacAddress": "02:42:ac:11:00:02",
                        "DriverOpts": null
                    }
                }

Test du port avec l'IP publique: 

    [root@786dac9969ee /]# curl http://172.17.0.2:4243
    /version
    {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"20.10.5","Details":{"ApiVersion":"1.41","Arch":"amd64","BuildTime":"2021-03-02T20:32:17.000000000+00:00","Experimental":"false","GitCommit":"363e9a8","GoVersion":"go1.13.15","KernelVersion":"4.19.128-microsoft-standard","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.4.3","Details":{"GitCommit":"269548fa27e0089a8b8278fc4fc781d7f65a939b"}},{"Name":"runc","Version":"1.0.0-rc92","Details":{"GitCommit":"ff819c7e9184c13b7c2607fe6c30ae19403a7aff"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"20.10.5","ApiVersion":"1.41","MinAPIVersion":"1.12","GitCommit":"363e9a8","GoVersion":"go1.13.15","Os":"linux","Arch":"amd64","KernelVersion":"4.19.128-microsoft-standard","BuildTime":"2021-03-02T20:32:17.000000000+00:00"}

----
## Jenkins Master:

Cette image est celle utilisée par le site officiel pour l'installation de Jenkins: [https://www.jenkins.io/doc/book/installing/docker/](https://www.jenkins.io/doc/book/installing/docker/)

Create a bridge network in Docker: `docker network create jenkins`
Build du Dockerfile se trouvant dans le dossier /jenkins-master/: `docker build -t jenkins .`
Run image for jenkins:  

    docker run -d -p 9090:8080 -p 50000:50000 -v jenkinsData:/var/jenkins_home jenkins/master
    a8db292de17bb44b40aa62d75fe7290ddb56cae0cd1df26355c2fb986f800142

Jump to the container:  `docker exec -it a8d /bin/bash`

Unlock Jenkins as usual:

    cat /var/lib/jenkins/secrets/initialAdminPassword

Browse to the  localhost:9090 and copy -paste the password into the inputfile


Vérification de la connexion avec l'hôte docker sur le master Jenkins:
    
    jenkins@a8db292de17b:/$ curl http://172.17.0.2:4243
    /version
    {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"20.10.5","Details":{"ApiVersion":"1.41","Arch":"amd64","BuildTime":"2021-03-02T20:32:17.000000000+00:00","Experimental":"false","GitCommit":"363e9a8","GoVersion":"go1.13.15","KernelVersion":"4.19.128-microsoft-standard","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.4.3","Details":{"GitCommit":"269548fa27e0089a8b8278fc4fc781d7f65a939b"}},{"Name":"runc","Version":"1.0.0-rc92","Details":{"GitCommit":"ff819c7e9184c13b7c2607fe6c30ae19403a7aff"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"20.10.5","ApiVersion":"1.41","MinAPIVersion":"1.12","GitCommit":"363e9a8","GoVersion":"go1.13.15","Os":"linux","Arch":"amd64","KernelVersion":"4.19.128-microsoft-standard","BuildTime":"2021-03-02T20:32:17.000000000+00:00"}

----

### Install plugin Docker cloud provider
on Jenkins interface : localhost:9090 

    > install plugin Docker cloud provider
    Add nodes > configure Cloud > add a new cloud: Docker + docker cloud details: 
    Name: Docker 
    Docker Host IP: tcp://172.17.0.2:4243 
    + test connection :
    + //return Version = 20.10.5, API Version = 1.41
    
    Check Enabled box
    Check Expose DOCKER_HOST box

Add Docker Template : 

    Labels: demo-docker-slave 
    check enabled box
    Name: demo-docker-slave
    
    Docker Image: bibinwilson/jenkins-slave
    
    Remote File System Root: /home/jenkins
    Connect method: Connect with SSH
    		select ssh credentials:
    # add credential : id docker-ssh | user: jenkins | mdp: jenkins 
    # and select this:
    		jenkins/****(docker-ssh)
    Host Key Verification Strategy: Non verification Strategy

nb: j'ai emprunté l'image du Hub de 
`bibinwilson/jenkins-slave` pour avoir un HubRepository (Docker Cloud... xP, merci bibin) 

----

### Configure a New Job

Got to the dashboard, on loaclhost:9090

    New item: docker slave demo
    Select 'Free style Project'
    	+ save

Configure pipeline: 

	   v Restreindre où le projet peut être exécuté	
	   Expression: demo-docker-slave 
    	/* return 
    	Label demo-docker-slave matches no nodes and 1 cloud. Permissions or other restrictions provided by plugins may further reduce that list.
    	*/

Ajouter une étape au build > Exécuter un script shell:  

    echo "this is run on Docker slave"

Apply and Save 

Click on the projet and Built it 

---

> it was not cake but amazing ;)