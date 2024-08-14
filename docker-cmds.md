# docker常用命令示例
## 在集群上部署应用
### 部署一个副本数量
```bash
#启动docker集群
docker service create --name myhello --punlish=81,target=80 --replicas 3 lirf233/hello:1.0.0
