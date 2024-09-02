# CI/CD & gitlab
## 1. 搭建私有注册中心
```bash
docker run -d -p 5000:5000 --name registry --restart=always registry:2
```
## 2. 修改每台主机的host文件
```bash
1.2.3.4 myhub.docker.com
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








