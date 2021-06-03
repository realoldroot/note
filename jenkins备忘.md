插件

[docker-plugin](https://plugins.jenkins.io/docker-plugin/)


### docker运行的jenkins想使用宿主主机的docker


修改docker-compose.yml，挂载进去下面目录

```
version: "3"
services:
  jenkins:
    image: jenkins/jenkins:lts-jdk11
    container_name: jenkins
    restart: on-failure
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /var/lib/docker:/var/lib/docker
    ports:
      - "7777:8080"
      - "50000:50000"
    deploy:
      resources:
        limits:
          cpus: "0.50"

networks:
  default:
    external:
      name: "my-net"

```

将/var/run/docker.sock用户改为Jenkins用户，不然Jenkins没权限执行docker
```
chown develop /var/run/docker.sock
```