## 快速入门指南

Open Horizon是一种中间件解决方案，旨在管理容器化应用程序的部署和服务软件生命周期。 
它还通过提供单独的模型同步服务来支持在服务和相关的机器学习资产之间进行可选的关注点分离。 
这些解决方案可以对处于各种连接状态的大量边缘计算节点进行自治管理。

要在一个设备上安装所有服务的简单且易于开发的版本，请在运行Ubuntu 18.04的x86_64计算机上以root身份运行以下单行代码：

```bash
curl -sSL https://raw.githubusercontent.com/open-horizon/devops/master/mgmt-hub/deploy-mgmt-hub.sh | bash
```