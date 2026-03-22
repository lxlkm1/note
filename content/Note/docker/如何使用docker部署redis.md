# 指令

```bash
sudo docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```

## 解释

### 大概

在服务器运行一个名为redis的redis-stack镜像容器，redis-stack 比 redis 增加了数据可视化界面，在localhost:8001打开设置用户密码，我们当前使用默认用户名default

### 详细


