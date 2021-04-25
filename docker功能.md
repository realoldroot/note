docker-compose加入已经存在的网络

```yaml
networks:
  default:
    external:
      name: my-pre-existing-network
```

docker创建网络

```
docker network create my-net        # 创建了一个名为"my-net"的网络
```

加入网络

```
docker network connect my-net test_demo         # 将Web服务加入my-net网络中
docker network connect my-net mysqld5.7         # 将mysql服务加入my-net网络中
```

断开网络

```
docker network disconnect bridge test_demo
docker network disconnect bridge mysqld5.7

```