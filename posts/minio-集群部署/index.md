# hugo 博客搭建



## 集群部署

1. 需要四个节点也就是四个服务器。
2. 四个硬盘（可以是假的，不过单独的硬盘对于恢复数据是比较好的）。

> docker compose 文件

```yaml
version: "3.8"

services:
  minio:
    image: minio/minio:RELEASE.2023-07-21T21-12-44Z
    environment:
      - "MINIO_ROOT_USER=roukaixin"
      - "MINIO_ROOT_PASSWORD=roukaixin"
    ports:
      - "9000:9000"
      - "9090:9090"
    volumes:
      - ./data/disk1/minio:/data/disk1/minio
      - ./data/disk2/minio:/data/disk2/minio
      - ./data/disk3/minio:/data/disk3/minio
      - ./data/disk4/minio:/data/disk4/minio
    command:
      - "server"
      - "http://minio{1...4}:9000/data/disk{1...4}/minio"
      - "--console-address"
      - ":9090"
    extra_hosts:
      - "minio1:10.1.1.1"
      - "minio2:10.1.1.2"
      - "minio3:10.1.1.3"
      - "minio4:10.1.1.4"
    network_mode: "host"
```

**注意：**

1.  `minio`  版本需要一致
2. **{1...4}** 表示的是一到四个节点。一到四之间是三个点。
3. **extra_hosts** 表示在容器主机内中的 `/etc/hosts` 文件中添加。
4. 一定需要使用 `host` 网络模式，如果不是 `host` 网络，那么启动就会报 `Unable to read 'format.json'` 错误

---

> 作者: pankx  
> URL: https://roukaixin.github.io/posts/minio-%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2/  

