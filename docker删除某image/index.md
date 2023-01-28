# docker删除镜像和容器



  
先查看容器运行状态

```
docker ps -a
```

找到image对应的容器，先stop
```
# id
docker stop d751d48a6d0a

```

再删除容器
```
docker rm d751d48a6d0a
```

最后才能删除镜像
```
 docker rmi fce289e99eb9
 
 
 # 查看镜像，删除ok
 docker images
```



- 注：rm与rmi的区别
```
rm Remove one or more containers
rmi Remove one or more images
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/docker%E5%88%A0%E9%99%A4%E6%9F%90image/  

