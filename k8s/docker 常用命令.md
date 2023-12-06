1.`docker stop `docker ps -f "name=^hf-mall_*"|awk '{if(NR>1){print $1}}'``<br />批量停止以`hf-mall` 开头名称的容器

```java
docker run -d --name mongodb -v c:/other/docker/mongodb/datadb:/data/db -p 27017:27017  --privileged=true mongo
```
