# docker常用命令示例
## 在集群上部署应用
### 1）简单部署
```bash
docker service create --name myhello --punlish=81,target=80 --replicas 3 lirf233/hello:1.0.0
```
### 2)部署带更新和回滚策略的应用
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
```bash
#检查策略是否配置成功
docker service inspect --pretty myhello1
#启动后更新策略配置
docker service update --update-delay 5s --rollback-monitor 10s myhello1
```
### 3)服务更新
```bash
docker service update --image lirf233/hello:1.0.1 myhello1
```