# minio 集群搭建



## 集群部署

1. 需要四个节点也就是四个服务器。
2. 四个硬盘（可以是假的，不过单独的硬盘对于恢复数据是比较好的）。

&gt; docker compose 文件

```yaml
version: &#34;3.8&#34;

services:
  minio:
    image: minio/minio:RELEASE.2023-07-21T21-12-44Z
    environment:
      - &#34;MINIO_ROOT_USER=roukaixin&#34;
      - &#34;MINIO_ROOT_PASSWORD=roukaixin&#34;
    ports:
      - &#34;9000:9000&#34;
      - &#34;9090:9090&#34;
    volumes:
      - ./data/disk1/minio:/data/disk1/minio
      - ./data/disk2/minio:/data/disk2/minio
      - ./data/disk3/minio:/data/disk3/minio
      - ./data/disk4/minio:/data/disk4/minio
    command:
      - &#34;server&#34;
      - &#34;http://minio{1...4}:9000/data/disk{1...4}/minio&#34;
      - &#34;--console-address&#34;
      - &#34;:9090&#34;
    extra_hosts:
      - &#34;minio1:10.1.1.1&#34;
      - &#34;minio2:10.1.1.2&#34;
      - &#34;minio3:10.1.1.3&#34;
      - &#34;minio4:10.1.1.4&#34;
    network_mode: &#34;host&#34;
```

**注意：**

1.  `minio`  版本需要一致
2. **{1...4}** 表示的是一到四个节点。一到四之间是三个点。
3. **extra_hosts** 表示在容器主机内中的 `/etc/hosts` 文件中添加。
4. 一定需要使用 `host` 网络模式，如果不是 `host` 网络，那么启动就会报 `Unable to read &#39;format.json&#39;` 错误

---

> 作者: 不北咪  
> URL: https://roukaixin.github.io/posts/2023-08/minio-%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2/  

