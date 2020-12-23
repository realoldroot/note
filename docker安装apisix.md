
# docker安装apisix和dashboard #

1. 拉取[apisix-docker](https://github.com/apache/apisix-docker)和[apisix-dashboard](https://github.com/apache/apisix-dashboard)源码。

2. 根据[官方指南](https://github.com/apache/apisix-dashboard/blob/master/docs/deploy-with-docker.md)在本机构建镜像文件。

    ```sh
    $ cd apisix-dashboard

    $ docker build -t apisix-dashboard:latest . --build-arg ENABLE_PROXY=true
    ```

3. 修改`apisix-dashboard/api/conf/conf.yaml`配置etcd地址。

    **文档上说只有`conf.listen.host=0.0.0.0`的时候，才可以通过外部网络访问容器内服务。**

    ```yaml
    conf:
    listen:
        host: 0.0.0.0 #这里修改了
        port: 9000
    etcd:
        endpoints:
        - 172.18.5.10:2379 #这里指向etcd地址。
    ```

    `172.18.5.10`这个ip是[官方例子](https://github.com/apache/apisix-docker/blob/master/example/docker-compose.yml)中配置的etcd地址。

4. 使用`apisix-docker`的例子来启动apisix，修改`apisix-docker/example/docker-compose.yml` 增加dashboard服务。

    ```yaml

    dashboard:
        image: apisix-dashboard:latest
        ports:
        - "9000:9000/tcp"
        volumes:
        - d:/project/apisix-dashboard/api/conf/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml
        #需要修改：映射到上面#3修改的apisix-dashboard/api/conf/conf.yaml
        networks:
        apisix:
            ipv4_address: 172.18.5.14

    ```


5. 运行服务，官方提供的apisix例子和dashboard。

    ```sh
    cd apisix-docker

    docker-compose -p docker-apisix up -d
    ```

6. 打开`http://localhost:9000/` 用户名密码是`admin` `admin`。



# 参考 

- [apisix-docker example](https://github.com/apache/apisix-docker/tree/master/example)

- [apisix-dashboard deploy-with-docker](https://github.com/apache/apisix-dashboard/blob/master/docs/deploy-with-docker.md)

- [Docker安装apisix](https://github.com/leon1509/note/blob/master/Docker%E5%AE%89%E8%A3%85apisix.md)
