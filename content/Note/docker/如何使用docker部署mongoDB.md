# 指令

```bash
sudo docker run -d \ 
--name mongo \ 
--restart unless-stopped \ 
--memory="8g" \ 
--cpus="4.0" \ 
-p 27017:27017 \ 
-v ~/docker/mongodb/data:/data/db \ 
mongo:latest
```

## 陷阱

docker 部署好的mongoDB 总是莫名其妙被关闭，原因是WiredTiger 存储引擎是mongoDB默认存储引擎,
WiredTiger 的默认内部缓存大小为以下两者中的较大值：
- 50%（RAM - 1GB）
- 0.256 GB。
我的电脑运行内存为32G,操作系统为arch linux,如果没有在docker 指定 mongo 最大可以访问的内存，mongo会占用我15GB,如果此时运行gnome桌面，chrome 浏览器，很容易导致OOM,linux 内核会杀死mongoDB,所以需要--memory参数指定mongoDB最大可以访问的内存
### 参考链接
1.[WiredTiger机制]([https://www.google.com](https://www.mongodb.com/docs/manual/core/wiredtiger/))

## 解释

### 大概

在服务器运行一个名为mongo的mongoDB镜像容器
### 详细
