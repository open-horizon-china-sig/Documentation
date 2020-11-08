## 安装Horizon代理(Agent)

### 虚拟机(VM)环境准备

建议您准备符合以下要求的虚拟机(VM)：

- 大于等于4GB内存
- 至少20GB储存空间
- Ubuntu Server 18.04 LTS

###依赖环境

如果未安装Docker或安装的版本早于18.06.01，请安装最新版本的Docker：
```bash
curl -fsSL get.docker.com | sh
```

###下载和安装示例
创建一个用于存放安装包的文件夹(目录可依据自己偏好更改)：
```bash
~$ mkdir -p pkgs
```
进入创建的文件夹:
```bash
~$ cd pkgs/
```
下载最新版的代理安装文件:
```bash
~/pkgs$ wget https://github.com/open-horizon/anax/releases/latest/download/horizon-agent-linux-deb-amd64.tar.gz
```
解压缩到当前文件夹:
```bash
~/pkgs$ tar -zxf horizon-agent-linux-deb-amd64.tar.gz
```
安装Horizon代理(Agent)和命令工具(CLI):
```bash
curl -fsSL get.docker.com | sh
```
验证代理是否正在运行：
```bash
hzn version
hzn exchange version
hzn node list
```
上述指令的输出应该与以下类似（版本号和URL可能不同）：
```bash
$ hzn version
Horizon CLI version: 2.23.29
Horizon Agent version: 2.23.29
$ hzn exchange version
1.116.0
$ hzn node list
{
      "id": "",
      "organization": null,
      "pattern": null,
      "name": null,
      "token_last_valid_time": "",
      "token_valid": null,
      "ha": null,
      "configstate": {
         "state": "unconfigured",
         "last_update_time": ""
      },
      "configuration": {
         "exchange_api": "https://9.30.210.34:8443/ec-exchange/v1/",
         "exchange_version": "1.116.0",
         "required_minimum_exchange_version": "1.116.0",
         "preferred_exchange_version": "1.116.0",
         "mms_api": "https://9.30.210.34:8443/ec-css",
         "architecture": "amd64",
         "horizon_version": "2.23.29"
      },
      "connectivity": {
         "firmware.bluehorizon.network": true,
         "images.bluehorizon.network": true
      }
   }
```
其他指令,如查看代理服务运行状态:
```bash
systemctl status horizon.service
```
###注册代理
TBD