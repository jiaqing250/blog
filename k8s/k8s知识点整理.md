### 1.三个端口号(nodePort、port、targetPort)的区别
- k8s服务的配置文件中几个端口参数，nodePort、port、targetPort，刚开始的时候不理解什么意思很容易混淆写错，这里总结一下，概括来说就是nodePort和port都是k8s的service暴露的端口，targetPort是容器本身暴露的端口。区别是nodePort暴露给k8s集群外部流量访问用，port暴露给k8s集群内部服务访问用。从上两个端口过来的数据最终都需要经过反向代理kube-proxy，流入后端pod的targetPort上，最后到达pod内的容器。

1、nodePort 端口<br />k8s集群中发发布完service之后，如果需要外部访问，nodePort是一种访问方式，即nodePort是提供给外部流量访问k8s集群中service使用的端口。例如外部用户要访问k8s集群中的一个Web应用，那么我们可以配置对应的service如下，就可以从外部通过浏览器http://node:28080访问到该web服务。<br />注意如果配置文件中不指定nodePort，部署后k8s会自动指定一个端口号来用。

2、port 端口<br />k8s集群内部服务之间相互访问service的端口。例如连接mysql使用3306端口，容器创建后暴露了3306端口，集群内其他容器想通过23306端口访问mysql服务，但是没有配置NodePort，外部流量就不能访问mysql服务。对应的service.yaml如下

3、targetPort端口<br />从上面例子也能看出来targetPort是什么，它就是容器真正暴露的端口（使用DockerFile中的EXPOSE），targetPort是pod上的端口，从port和nodePort上来的流量，经过kube-proxy流入到后端pod的targetPort上，最后进入容器内。例如一个容器暴露8080端口的tomcat完整配置如下，nodePort可以不配，会自动指定一个端口。

2.就绪探针和存活探针
```yaml
		readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

参考资料: [https://jimmysong.io/kubernetes-handbook/guide/configure-liveness-readiness-probes.html](https://jimmysong.io/kubernetes-handbook/guide/configure-liveness-readiness-probes.html)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/2923644/1622788379450-466d4801-d954-444b-8b5a-b5c5e0dad2b9.png#averageHue=%232d2c2b&height=262&id=zDcnx&originHeight=523&originWidth=673&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32551&status=done&style=none&title=&width=336.5)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/2923644/1622788423053-e60ef30b-975c-4fe0-8084-76e2f4d5849b.png#averageHue=%23b9ddeb&height=341&id=VBP6c&originHeight=682&originWidth=1157&originalType=binary&ratio=1&rotation=0&showTitle=false&size=80307&status=done&style=none&title=&width=578.5)
