# docker笔记
## 常用网站
> - https://github.com/widuu/chinese_docker
> - http://dockone.io/
> - https://www.daocloud.io/
> - http://www.dockerinfo.net/


## 教程
> - build镜像 https://docs.docker.com/engine/tutorials/dockerimages/
> - 可视化dock data https://github.com/justone/dockviz
> - dao cloud 文档 http://docs.daocloud.io/quick-start

## 镜像库
> - https://hub.docker.com/explore/

## 自己build镜像
> * pull镜像, docker pull centos:6.7
> * 运行docker, docker run -ti centos:6.7 /bin/bash  i,交互,t为终端,执行bash
> * 运行其它的东西
> * 断线之后再次进入docker,docker attach exec nsenter nsinit 
```javascript
docker ps
docker ps --no-trunc
docker inspect image_id
docker attach image_id
docker exec -it image_id /bin/bash
```
> * 登录docker hub  

```java
docker login --username=jy0hub --email=mylijinya@gmail.com
```
> * 提交image
```c
查看配置
 .docker/config.json 
 { 
    "auths": { 
        "https://index.docker.io/v1/": { 
            "auth": "ankwaHViOmxqeSNodWI="
        }
    }
}
docker commit -m="add gcc in centos 6.7" -a="ya jin" 1f3b93950104 jy0hub/centos
删除镜像  docker rmi  e10b117bd7b6
向hub push自己的镜像 docker push jy0hub/centos
```
## slack使用docker
```javascript
docker -d -s overlay -D
```
