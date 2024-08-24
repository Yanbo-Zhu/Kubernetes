
# 1 题设


一个Dockerfile 已经存在于 **/ckad/DF/Dockerfile**
1. 使用已存在的 Dockerfile ，构建一个名为 **centos**和标签为 **8.2** 的容器镜像。您可以安装和使用您选择的工具。
2. 使用您选择的工具，以 OCI 格式导出构建的容器镜像，并将其存储在 **/ckad/DF/centos-8.2.tar**

# 2 参考 

参考docker 构建
https://docs.docker.com/language/java/build-images/


# 3 解题 

1.查看dockerfile文件，根据要求构建镜像
/ckad/DF/Dockerfile

有可能要修改dockerfile文件 (这步可以省略)
```
FROM centos:8
LABEL maintainer="test dockerfile"
LABEL test=dockerfile
USER root
RUN useradd shadow
RUN mkdir /opt/shadow
```

2 构建镜像 
cd /ckad/DF/    # 要切换到 Dockerfile 文件所在的目录做镜像
sudo docker build -t centos:8.2 .   # 注意命令最后还有一个小数点，表示当前目录。
sudo docker images

检查镜像
sudo docker images | grep centos | grep 8.2

3 导出镜像到文件：/ckad/DF/centos-8.2.tar
sudo docker save centos:8.2 > /ckad/DF/centos-8.2.tar

检查存储的镜像
ll /ckad/DF/centos-8.2.tar






