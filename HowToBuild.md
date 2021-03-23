# `Dapr-dashboard` 编译
dapr-dashboard 分两部分，前端 nodejs 和后端go，go部分可以在 mips 上编译出来，但是 nodejs 部分因为缺少包过不了，但是缺少的包不影响最终结果，所以可以在 x86 下编译 nodejs部分，在龙芯上编译go 部分，然后二者结合制作镜像

#第一阶段 在x86 上编译
`os: ubuntu 18.04 x64`
为了方便这里采用docker + 宿主机的模式编译。整个编译在docker 环境中进行，在编译完之后制作docker 镜像时在宿主机中进行。

## 在宿主机
```
apt install docker.io
systemctl start docker
docker pull ubuntu

docker run -itd -v /home/vm/work:/mnt --name ubuntu ubuntu /bin/bash

docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ed1615701f8a        ubuntu              "/bin/bash"         11 minutes ago      Up 11 minutes                           ubuntu

# 进入docker 环境
docker exec -it ed1615701f8a /bin/bash
```
## 在docker 环境
```
apt install build-essential
apt install git
apt install golang-go
apt install npm
apt install docker.io

cd /mnt
tar xf dapr-dashbard-0.6.0.tar.gz
cd dashbard-0.6.0
export GOPROXY="https://goproxy.io"

#第一次编译生成 ng 
make

# 设置ng 到环境变量中
cd ./web/node_modules/@angular/cli/bin
export PATH=$PATH:`pwd`
cd -

make
```

## 在宿主机 (如不想在本机制作镜像此步骤可以省略)
```
cd /home/vm/work/dapr-dashbard
make docker-build DAPR_REGISTRY=0.6.0 DAPR_TAG=amd64

fatal: not a git repository (or any of the parent directories): .git
Building 1.0.0/dashboard:amd64 docker image ...
docker build --build-arg BIN_PATH=release/linux_amd64 -f ./docker/Dockerfile . -t 1.0.0/dashboard:amd64-linux-amd64 --platform linux/amd64
"--platform" is only supported on a Docker daemon with experimental features enabled
docker/docker.mk:65: recipe for target 'docker-build' failed
make: *** [docker-build] Error 1

vim /etc/docker/daemon.json
{
"experimental": true
}
systemctl restart docker
```

## 在宿主机环境将编译完成的打包
```
tar zcf dashbard-0.6.0-compile.tar.gz dashbard-0.6.0
```


# 第二阶段在龙芯平台编译
`os: Loongnix Server 1.7`

```
tar xf dashbard-0.6.0-compile.tar.gz 
cd  dashbard-0.6.0
export GOPROXY="https://goproxy.io"
# 应用补丁
patch -p0 < 0001-add-mips-support.patch

# 设置ng 到环境变量中
cd ./web/node_modules/@angular/cli/bin
export PATH=$PATH:`pwd`
cd -

make -f .loongson/Makefile
```

