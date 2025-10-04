## 简介
A set of shell scripts that will build GNU/Linux distribution rootfs image
for rockchip platform.

我目前只使用docker去进行构建，没有直接使用宿主机，所以这里只有docker构建的步骤

## 适用板卡
- 使用RK3566处理器的泰山派板卡(不支持gnome桌面)

## 前置依赖
首先使用给定的dockerfile去构建docker镜像
```
docker build -t lubancat_build_rootfs:v2 . -f Docker/Dockerfile
```

## 在docker容器中构建 Ubuntu22.04镜像（仅支持64bit）步骤
- lite：控制台版，无桌面
- xfce：桌面版，使用xfce桌面套件
- xfce-full：桌面版，使用xfce桌面套件+更多推荐软件包
- gnome：桌面版，使用gnome桌面套件
- gnome-full：桌面版，使用gnome桌面套件+更多推荐软件包

### 启动docker容器
```
docker run --privileged -d \
--name lubancat_build_ubuntu \
--hostname build_ubuntu \
-v /root/new_workspace/tspi_ubuntu:/ubuntu \ # 这里/root/new_workspace/tspi_ubuntu换成你们本地的仓库根路径
-it lubancat_build_rootfs:v2 /bin/bash
```

### 进入容器
```
docker ps|grep lubancat_build_ubuntu # 查看容器id，输出的第一列就是
docker exec -it ${容器id} /bin/bash
```

### 进入容器后执行
#### step1.准备
```
cd /ubuntu
# 注册binfmts，在x86的容器中，执行跨架构命令依赖这个，如果是arm的容器，应该就不需要
update-binfmts --enable qemu-aarch64
# 安装依赖
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f
```

#### step2.构建基础 Ubuntu 系统。

```
# 运行以下脚本，根据提示选择要构建的版本
./mk-base-ubuntu.sh
```

#### step3.添加 rk overlay 层,并打包ubuntu-rootfs镜像

```
# 运行以下脚本，根据提示选择要构建处理器版本和ubuntu的版本
./mk-ubuntu-rootfs.sh
```