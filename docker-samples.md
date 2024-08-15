# docker部署示例
## 在集群上部署应用
### 1）简单部署
```bash
docker service create --name myhello --punlish=81,target=80 --replicas 3 lirf233/hello:1.0.0
```
### 2) 部署带更新和回滚策略的应用
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
### 3) 服务更新
```bash
docker service update --image lirf233/hello:1.0.1 myhello1
```
### 4) 服务回滚
```bash
docker service update --rollback myhello1
```
## 结合docker-compose.yml在集上群部署应用
### docker-compose.yml文件
```yml
version: "3.7"
services:
    myhello2:
        image: lirf233/hello:1.0.0
        ports:
            - "83:80"
        depends_on:
            - redis
        deploy:
            mode: replicated
            replicas: 20
            endpoint_mode: vip
            rollback_config:
                parallelism: 2
                delay: 10s
                monitor: 10s
                max_failure_ratio: 0.2
            update_config:
                parallelism: 2
                delay: 5s
                failure_action: continue
    redis:
        image: redis:alpine
        deploy:
            mode: replicated
            replicas: 6
            endpoint_mode: dnsrr
            labels:
                description: "This redis service label"
            resources:
                limits:
                    cpus: '0.50'
                    memory: 50M
                reservations:
                    cpus: '0.25'
                    memory: 20M
                restart_policy:
                    condition: on-failure
                    delay: 5s
                    max_attempts: 3
                    window: 120s
```
