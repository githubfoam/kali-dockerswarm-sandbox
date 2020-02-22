# kali-dockerswarm-sandbox

master switches
~~~
kali-docker-swarm-sandbox-centos\Vagrantfile

auto = ENV['AUTO_START_SWARM'] || true # create swarm auto mode
multi_manager =  true # create swarm multi manager mode
~~~
multi-master mode
~~~
>vagrant up
>vagrant ssh manager

    [vagrant@manager ~]$ hostnamectl
       Static hostname: manager
             Icon name: computer-vm
               Chassis: vm
            Machine ID: 2eb2b7d854e94f58b4899411ef23ffe5
               Boot ID: ba205d3c06134ae3821153d373fed9e6
        Virtualization: kvm
      Operating System: CentOS Linux 7 (Core)
           CPE OS Name: cpe:/o:centos:centos:7
                Kernel: Linux 3.10.0-1062.9.1.el7.x86_64
          Architecture: x86-64

      [vagrant@manager ~]$ cat /etc/redhat-release
      CentOS Linux release 7.7.1908 (Core)
      [vagrant@manager ~]$ whoami
      vagrant
      [vagrant@manager ~]$ id vagrant
      uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),993(docker)

vagrant@manager:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
kscv757m5li5u3vkdy6ogj93h *   manager             Ready               Active              Leader              19.03.2
upkfbsb1cbbddfc7bbznqyze1     manager1            Ready               Active              Reachable           19.03.2
uxalnnzeegjzz6ep7549vyigc     worker1             Ready               Active                                  19.03.2
apxj1nh24lxxeqjul2nij5edh     worker2             Ready               Active                                  19.03.2

~~~

local docker registry
~~~
docker run -d   -p 5000:5000   --name registry   -v /opt/registry/data:/var/lib/registry   --restart always   registry:2

docker build . -t kali:cmd --file /vagrant/kalilinux/Dockerfile
docker tag kali:cmd localhost/kali:cmd
docker tag localhost/kali:cmd localhost:5000/kali:cmd

docker push localhost:5000/kali:cmd

docker image rm localhost:5000/kali:cmd
docker image rm localhost/kali:cmd

docker pull localhost:5000/kali:cmd

~~~
~~~

[vagrant@manager ~]$ docker service create --name registry --publish published=5000,target=5000 registry:2
0vjes7ujrk4ix2eux0m96ru3u
overall progress: 1 out of 1 tasks                                                                                                                                                           1/1: running   [==================================================>]                                                                                                                         verify: Service converged                                                                                                                                                                    

[vagrant@manager ~]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
0vjes7ujrk4i        registry            replicated          1/1                 registry:2          *:5000->5000/tcp

[vagrant@manager ~]$ curl http://localhost:5000/v2/
{}

  [vagrant@manager ~]$ docker-compose -f /vagrant/kalilinux/docker-compose.yml up -d
  WARNING: The Docker Engine you're using is running in swarm mode.

  Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.

  To deploy your application across the swarm, use `docker stack deploy`.

  Creating kalilinux_kali-service_1 ... done

  [vagrant@manager ~]$ docker-compose -f /vagrant/kalilinux/docker-compose.yml ps
            Name              Command    State    Ports
  -----------------------------------------------------
  kalilinux_kali-service_1   /bin/bash   Exit 0

  [vagrant@manager ~]$ docker stack deploy  --compose-file /vagrant/kalilinux/docker-compose.yml kalilinux_kali-service_1
  Creating network kalilinux_kali-service_1_default
  Creating service kalilinux_kali-service_1_kali-service

  [vagrant@manager ~]$ docker stack services kalilinux_kali-service_1
  ID                  NAME                                    MODE                REPLICAS            IMAGE                     PORTS
  ak8qn6c95cut        kalilinux_kali-service_1_kali-service   replicated          0/1                 localhost:5000/kali:cmd

  [vagrant@manager ~]$ docker stack ls
  NAME                       SERVICES            ORCHESTRATOR
  kalilinux_kali-service_1   1                   Swarm

~~~
