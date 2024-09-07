# CI/CD & gitlab
## 1. 搭建私有注册中心
```bash
docker run -d -p 5000:5000 --name registry --restart=always registry:2
```
## 2. 修改每台主机的host文件
```bash
192.168.x.x myhub.docker.com
```
## 3. 修改每台主机的docker配置文件
```bash
vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry myhub.docker.com:5000
#追加的内容： --insecure-registry myhub.docker.com:5000
sudo systemctl daemon-reload
sudo systemctl restart docker
```
## 4. 将所需要的镜像拉取至私有注册镜像仓库
```bash
docker pull golang:1.18
docker pull docker:20.10.12
docker pull alpine:latest
docker pull ubuntu:latest
docker pull gitlab/gitlab-ce:latest
docker pull gitlab/gitlab-runner:latest

docker tag golang:1.18 myhub.docker.com:5000/golang:1.18
docker tag docker:20.10.12 myhub.docker.com:5000/docker:20.10.12
docker tag alpine:latest myhub.docker.com:5000/alpine:latest
docker tag ubuntu:latest myhub.docker.com:5000/ubuntu:latest
docker tag gitlab/gitlab-ce:latest myhub.docker.com:5000/gitlab-ce:latest
docker tag gitlab/gitlab-runner:latest myhub.docker.com:5000/gitlab-runner:latest

docker push myhub.docker.com:5000/golang:1.18
docker push myhub.docker.com:5000/docker:20.10.12
docker push myhub.docker.com:5000/alpine:latest
docker push myhub.docker.com:5000/ubuntu:latest
docker push myhub.docker.com:5000/gitlab-ce:latest
docker push myhub.docker.com:5000/gitlab-runner:latest
```
## 5. 启动gitlab-ce
```bash
docker run -d --hostname 192.168.x.x -p 80:80 --name gitlab-ce myhub.docker.com:5000/gitlab-ce:latest 
```
## 6. 查看容器初始密码并登录修改
```bash
docker exec -it gitlab-ce grep 'Password:' /etc/gitlab/initial_root_password
```
## 7. 运行gitlab-runner容器并绑定宿主机socket
```bash
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
myhub.docker.com:5000/gitlab-runner:latest
```
## 8. 注册gitlab-runner
```bash
docker run --rm -it -v /src/gitlab-runner/config:/etc/gitlab-runner myhub.docker.com:5000/gitlab-runner register
#填写gitlab实例
#http://192.168.254.131
#http://192.168.254.135
#http://192.168.254.139
#输入runner注册码
#输入描述信息，举例：The builder，The tester，The deployer等；
#输入tag，举例：builder，tester，deployer等；
#输入maintenance运维说明；
#executor 选择 docker
#default Docker image选择：myhub.docker.com:5000/ubuntu:latest
```
## 9. 修改gitlab-runner在宿主机上的配置
```bash
#sudo nvim /srv/gitlab-runner/config/config.toml
volumes = ["/var/run/docker.sock:/var/run/docker.sock","/cache"]
```


## 以下为实际部署时候的情况，供参考

### *. 零声教程（不一定正确，仅供参考
```bash
docker run -d --hostname 192.168.254.135 \
-p 443:443 -p 80:80 --name gitlab-ce \
--restart always \
-v $HOME/work/gitlab/config:/etc/gitlab \
-v $HOME/work/gitlab/logs:/var/log/gitlab \
-v $HOME/work/gitlab/data:/var/opt/gitlab \
--shm-size 256m \
gitlab/gitlab-ce:16.2.4-ce.0

docker pull quay.io/0voice/gitlab/gitlab-runner:v16.2.1

docker run -d --name gitlab-runner1 --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:v16.2.1


docker exec gitlab-runner1 gitlab-runner -v

gitlab-runner register  --url http://192.168.254.135 \
--token glrt-ACXbFYRcw1z9xYsU8_kX \

docker run --rm -it -v /src/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:v16.2.1 register \
-n \
--url "http://192.168.254.135" \
--token "glrt-ACXbFYRcw1z9xYsU8_kX" \
--name "gitlab-runner1" \
--executor docker \
--docker-image docker:24.0.5 \
--docker-volumes /var/run/docker.sock:/var/run/docker.sock \
--docker-pull-policy if-not-present
```
### 解决连接不上runner的解决方法

 https://gitlab.com/gitlab-org/gitlab-runner/-/issues/29285

```bash
docker run gitlab/gitlab-runner verify
```
```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```
```bash
docker run --rm -it -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
```
