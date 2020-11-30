##本地开发
本地开发环境搭建

###e2edev开发/测试环境介绍
e2edev(End to end Horizon developer test)是用来给Open Horizon开发人员测试X86设备和X86 Agbot，
因此也叫End to End test。

此开发环境会创建3个容器：

- exchange-db :用于exchange-api的postgres容器
- exchange-api :从openhorizon/amd64_exchange-api：latest中pull的exchange-api容器
- agbot :最初是从openhorizon/anax源构建的运行anax （Horizon Agent控制系统）的容器。除非另有说明，
  否则将使用源的本地副本，并且根据选择的PATTERN创建一系列工作负载容器。
 
###虚拟机(VM)环境准备
建议您准备符合以下要求的虚拟机(VM)：
- 大于等于4GB内存
- 至少20GB储存空间
- Ubuntu Server 18.04 LTS

###构建和运行
一步一步设置您的环境 

安装docker:
```bash
curl https://get.docker.com/ | sh
```
安装make和jq:
```bash
apt update && apt install -y make jq build-essential
```
安装golang版本1.14.*:
```bash
curl https://dl.google.com/go/go1.14.linux-amd64.tar.gz | tar -xzf- -C /usr/local/
export PATH=$PATH:/usr/local/go/bin (and modify your ~/.bashrc file with the same)
```
配置GOPATH和ANAX_SOURCE路径，注意无法将GOPATH设置为与GOROOT相同的路径:
```bash
export GOPATH=</your/go/path> (typically $HOME/go)
export ANAX_SOURCE=</path/to/anax>
```
设置单个节点[microk8s](https://microk8s.io/docs/)进行测试，
注意要通过1.18 channel安装microk8s。另请参阅https://microk8s.io/docs/setting-snap-channel升级microk8s:
```bash
sudo snap install microk8s --classic --channel=1.18/stable
```
克隆anax的git仓库:
```bash
git clone git@github.com:open-horizon/anax.git
```
构建anax二进制文件和agbot基本镜像，并提取exchange组件镜像：
```bash
(cd .. && make)
```
构建e2edev Docker镜像
```bash
make
```

###开发迭代-基本操作
这是最全面的“基本”测试：

- 它会创建agbot，Exchange-db和exchange-api容器，并复制所有anax / exchange配置和二进制文件。
- 它将生成协议（agreement）并运行所有工作负载，确认已通过的协议，保留协议，成功取消协议并重新制定协议。
- 它每次运行都是基于一个全新的环境
```bash
make test
```

###开发迭代-高级操作
您可以在make run-combined命令上指定几个环境变量，以决定e2edev环境中运行的测试用例。
在开发过程中运行环境的常见方法是：
```bash
make test TEST_VARS="NOLOOP=1 PATTERN=sloc"
```
轻量级测试：
```bash
make test TEST_VARS="NOLOOP=1 NOCANCEL=1 NOHZNREG=1 NORETRY=1 NOSVC_CONFIGSTATE=1 NOSURFERR=1 NOUPGRADE=1 NOPATTERNCHANGE=1 NOCOMPCHECK=1"
```
以下是所有测试变量的完整说明：

| 参数| 说明 |
| ------------- | ------------- |
| NOLOOP = 1  | 每10分钟关闭取消设备和agbot（交替）协议的循环。通常，在主动迭代代码时，您需要指定NOLOOP = 1。 |
| NOCANCEL = 1  | 当设置为NOLOOP = 1时，仅对协议形成感兴趣时，跳过单轮取消测试，以减少日志混乱和时间。  |
| UNCONFIG = 1 | 打开unconfig/reconfig循环测试。|
| PATTERN = name|指定您希望设备使用的配置模式的名称。内置模式为spws，sns，sloc，sgps，sall，cpu2msghub等。 如果指定PATTERN，但关闭了最重要服务所需的从属服务之一，则系统将无法正常工作。如果您未指定PATTERN，则将使用手动管理的策略文件来运行工作负载（除非您将其关闭）。|
| NOHZNREG = 1|使用hzn命令关闭用于注册/注销节点的测试。|
| NORETRY = 1|关闭服务重试测试。|
| NOSVC_CONFIGSTATE = 1|关闭服务配置状态测试。|
|NOSURFERR = 1|关闭节点表面错误测试。|
|NOUPGRADE = 1|关闭服务升级/降级测试。|
|NOPATTERNCHANGE = 1|关闭节点模式更改测试。|
|NOCOMPCHECK = 1|关闭策略兼容性测试。|
|NOSDO = 1|关闭SDO测试。|
|NONS = 1|不注册netspeed服务。|
|NOGPS = 1|不注册gpstest服务。|
|NOLOC = 1|不注册位置服务。|
|NOPWS = 1|不注册天气服务。这是迭代代码时要运行的一个很好的工作负载，因为它简单可靠，不会妨碍您的工作。|
|NOK8S = 1|不注册k8s-service1。|
|NOANAX = 1|anax已启动以进行API测试，但随后停止了，并且没有重新启动以运行工作负载。|
|NOAGBOT = 1|永远不会启动agbot。|
|HA = 1|将2台设备（和工作负载服务）注册为HA对。您将在容器中获得2个anax设备进程。|
|OLD_ANAX = 1|根据github中的当前提交运行anax设备，即在您进行更改之前运行该设备。这对于新agbot与先前设备的兼容性测试非常有用。|
|OLD_AGBOT = 1|在进行更改之前，基于github中的当前提交运行agbot，即agbot。这对于新设备与以前的agbot的兼容性测试很有帮助。|

###调试相关信息
####进入agbot容器
```bash
docker exec -it agbot /bin/bash
```
- 所有日志文件都位于agbot容器的`/tmp`目录下。 `/tmp/anax.log`用于记录设备有关日志，`/tmp/agbot.log`用于记录agbot相关日志。
- 重要数据文件和脚本位于`/root/`,`/root/.colonus`和`/root/eth`中
- 配置文件在`/etc`中

####在容器外
从设备的角度查看所有当前和已存档的协议(agreement):
```bash
curl http://localhost/agreement | jq -r '.agreements'
```
从agbot的角度向您显示所有当前和已存档的协议
```bash
curl http://localhost:81/agreement | jq -r '.agreements'
```
访问位于http://localhost:8080/v1的Exchange API文档

###Clean选项/开发人员流程
删除工作负载，agbot/exchange-api/db容器，exchange中的所有数据以及所有过时的配置/脚本...所有在`make test`中自动运行的文件使用:
```bash
make clean
```
如`make clean`一样删除所有内容并删除anax二进制文件（用于更改anax）
```bash
make mostlyclean
```
完成上述所有操作且删除agbot和exchange基本镜像，docker测试网络以及所有闲置的docker镜像\
注意：这是唯一需要重新运行make的"clean"命令
```bash
make realclean
```
###远程环境测试
TBD