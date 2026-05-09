
# 概述

docker 是一个开发快速部署环境的工具，可以保证多个电脑的开发环境一致

# 常用指令


## 查看状态

```bash
docker ps #查看现在在运行的容器
docker ps -a #查看所有容器，包括没有在运行的
docker images #查看现在有的镜像
```

## 拉取镜像

```bash
docker pull [image name] # name:version
```

## 运行镜像

```bash
sudo docker run -d \ 
--name [container name] \ 
--restart [time to restart] \ 
--memory=[max use memory for docker container] \ 
--cpus=[max use cpu cores for docker container] \ 
-p [real compoter port]:[docker compoter port] \ 
-v [real compoter disk]:[docker compoter disk]\ 
[image name] #mongo:latest  
```

## 删除容器

```bash
sudo docker stop [container name] #在删除容器之前必须先停止容器
sudo docker rm [container name or id]
```

## 查看日志

```bash
sudo docker logs [container name] | grep -i "error"
```


## 问题解决

### docker 和 docker desktop 容器状态不一致


### docker 导致服务器在10.102.4.110 类似的局域网无法被访问