# docker常用命令示例
## 在集群上部署应用
### 1）部署一个副本数量
```bash
#启动docker集群
docker service create --name myhello --punlish=81,target=80 --replicas 3 lirf233/hello:1.0.0
```
### 2)在集群上部署一个带更新和回滚策略的应用
```bash
docker service create \
--name myhello1 \
--publish published=82,target=80 \
--replicas 20 \
--update-delay 5s \
--update-parallelism 2 \
--update-failure-action continue \
--rollback-parallelism 2 \
--rollback-monitor 10s \
--rollback-max-failure-ratio 0.2 \

lirf233/hello:1.0.0
```