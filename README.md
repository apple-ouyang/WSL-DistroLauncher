# 如何在WSL运行openEuler

1. 使用我构建好的启动包
2. 手动导入openEuler的docker镜像

## 使用我构建好的启动包

由于该UWP还没有上传到微软商店，因此暂时只能在本地打开，有2种方法：

1. 双击运行 x64\Debug\launcher.exe，安装openEuler
2. 或者按照以下步骤构建安装包。请注意，我还没有在其他电脑上测试，不确定我生成的签名在其他电脑上是否会导致错误

### 使用工程文件构建启动包

使用Visual Studio打开WSL-DistroLauncher工程下的DistroLauncher.sln

在Solution Explorer，一般在右侧，可以看到以下界面

![Solution](./readme_img/Solution.png)

右键点击"Solution (DistroLauncher)"，在弹出菜单中点击Deploy Solution

等待编译完成后，即可在左下角Windows开始菜单中启动openEuler

![image-start_menu](./readme_img/start_menu.png)

点击即可运行openEuler

或者命令行运行：

```
wsl -d openEuler
```

![image-openEuler_in_wsl](./readme_img/openEuler_in_wsl.png)

## 手动导入openEuler

参考文档：https://docs.microsoft.com/en-us/windows/wsl/use-custom-distro

下载openEuler LTS的补丁镜像中的docker镜像：https://repo.openeuler.org/openEuler-20.03-LTS-SP1/docker_img/x86_64/openEuler-docker.x86_64.tar.xz



**在下载的镜像所处文件夹下启动WSL下的Ubuntu**，此时Ubuntu工作目录为当前目录

```
wsl -d Ubuntu
```



安装Ubuntu下的docker

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```



导入镜像，-i表示使用tar包来导入镜像

```shell
docker load -i .\openEuler-docker.x86_64.tar.xz
```



查看现有images

```
docker images
```



应该会有以下输出：

```
REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE
openeuler-20.03-lts-sp1    latest    6934cec25f28   3 months ago    512MB
```



随便运行一个命令，加载容器

```sh
docker run openeuler-20.03-lts-sp1 echo hello, openEuler WSL
```



导出docker容器的快照，即当前文件系统。

docker ps -ql表示获得最近运行的容器的编号，即刚才openEuler的容器编号

```shell
docker export $(docker ps -ql) > ./openEuler.tar
```



退出Ubuntu，启动WSL，导入openEuler包。

设置D:\work\WSL\openEuler为WSL的安装目录

```
wsl --import openEuler D:\work\WSL\openEuler .\openEuler.tar
```



即可启动openEuler

```
wsl -d openEuler
```



## 构建launcher.exe的过程

1. 克隆官方启动器的仓库，修改启动器基本信息：使用Visual Studio，修改软件包名称，发布公司，修改图片logo等。
2. 安装WSL、Ubuntu
3. 导出docker：在Ubuntu下使用load命令载入docker镜像，使用export导出容器当前文件系统快照，压缩为install.tar.gz
4. 构建包：将install.tar.gz，放入工程项目根目录下
5. 使用VS，构建项目解决方案
6. 在windows开始菜单中出现openEuler图标，点击即可运行openEuler-WSL

## 详细过程

### 克隆，修改基本信息

```shell
git clone https://github.com/Microsoft/WSL-DistroLauncher
```



首先安装Visual Studio，VS code好像没有下面会用到的功能

在Visual Studio中打开项目文件

双击打开MyDistro.appxmanifest，此时VS会自动探测xml格式，并出现很好看的修改界面如下。

![image-20210412172319111](./readme_img/packaging.png)

如果是纯xml文件，需要配置相关环境，这里我百度卡了很久，具体解决方法忘了，好像是要VS中装C++桌面开发需要的SDK



在该的界面上，点击Packaging选项卡，点击Choose Certificate...，点击Create...，这里输入Publish Name，我输入了Huawei，本来想用全称HUAWEI TECHNOLOGIES CO., LTD，但是会报错，就用了Huawe。然后输入密码，即可创建证书。



修改Application选项卡下的基本信息，修改Visual Asserts图片信息。

图片信息可以使用Asserts Generator，在基于一个logo的图片下，生成不同大小的格式的图片。

官网的logo太小，我找到了部门里的logo矢量图.ai文件，放大了些，并参考Ubuntu启动图标，裁剪了文字部分，只保留了logo，尽量让产生的logo在启动界面好看一些。最后生成的所有图片可以看我的工程文件。

### 安装WSL、Ubuntu

安装WSL可以参考官方文档，https://docs.microsoft.com/en-us/windows/wsl/install-win10

然后应用商店安装Ubuntu，运行后自动创建root用户，输入用户名密码即可

### 导出docker

[参考手动导入一节](# 手动导入openEuler)

最后一步改为

```
docker export $(docker ps -ql) > ./install.tar
```

退出Ubuntu，压缩刚才的包

```
exit
gzip.exe -k .\install.tar
```

-k表示保留包，不删除

### 构建包

将install.tar.gz复制到项目的根目录下

右键点击VS解决方案目录下的"Solution (DistroLauncher)"，点击Deploy Solution

等待即可构建完成

