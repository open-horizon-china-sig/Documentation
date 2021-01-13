##SDO支持
###SDO介绍
自动化的“零接触”安裝服务
为了更安全，自动地安装和配置边缘设备，只需要将其直接运送到安装点，连接到网络并加电即可。 SDO负责其余的工作。这种零接触模型简化了安装人员的角色，降低了成本，并消除了不良的安全做法，例如传输默认密码。

Secure Device Onboard可以更轻松，更快，更便宜，更安全地启动设备。它扩展了物联网设备的TAM，进而加速了数据处理基础架构的生态系统。大多数“零接触”自动入门解决方案都需要目标平台由制造商决定。

Secure Device OnBoard的主要优点包括：

1. 启用按计划构建模型-ODM可以使用标准化制造工艺来大批量构建相同的IOT设备。减少库存，供应周期时间和成本。
2. SDO“后期绑定” –允许在第一次开机时在供应链中“后期”选择设备的目标平台。
3. 它是开放的-意味着其服务和云独立。设备在安装时必然会针对生态系统。与现有的云服务一起使用，不会替代它们。

####起源与历史
英特尔公司于2020年2月基于英特尔®SDO版本1.8将Secure Device OnBoard作为开源软件发布。

最初的英特尔®SDO作为独立的英特尔产品于2017年9月推出，反映了最初的SDO协议和体系结构规范。凭借此产品成功所需的复杂生态系统，英特尔决定开源并向社区捐赠英特尔®SDO的核心功能，以推动行业标准，解决关键的行业摩擦点并允许物联网市场更快地增长。英特尔相信，具有生机勃勃的生态系统的开放式采购将使SDO演变成真正的行业标准。

####跨越边缘整合
机载安全设备的主要目标是为物联网设备扩展TAM。为了实现这一目标，需要设备制造商，分销商，系统集成商，云服务提供商和设备管理软件供应商的跨行业协作以加快采用速度。LinuxFoundation的LF Edge是促进这种协作并加快采用速度的理想组织。这一重要技术。

SDO将加速设备在家庭和工业生态系统中的采用，从而有助于推动LFEdge社区中所有当前项目的需求。
###Open Horizon SDO支持概述
只需导入关联的所有权凭证，然后打开设备电源，即可将使用[Intel SDO](https://software.intel.com/content/www/us/en/develop/tools/secure-device-onboard.html)（安全设备板载）构建的边缘设备添加到Open Horizon实例中。

Open Horizon为了集成SDO，对SDO做了定制化处理。详见Open Horizon的[SDO-support](https://github.com/open-horizon/SDO-support) Github库。

该Github存储库中的软件提供了SDO和Open Horizon之间的集成，从而使在Horizon中轻松使用支持SDO的边缘设备。 Horizon SDO支持包括以下组件：

所有SDO“所有者”服务（在Horizon管理中心上运行的服务）的合并docker映像。
SDO集合服务器（RV）是引导启用SDO的设备启动的第一个服务。集合服务器将设备重定向到正确的Horizon实例以进行配置。
名为generate-key-pair.sh的脚本可自动执行创建私钥，证书和公钥的过程，因此每个租户都可以导入和使用自己的密钥对，以便他们可以安全地使用SDO。
hzn voucher子命令，用于将一个或多个所有权凭证导入到horizon实例中。（所有权凭证是设备制造商与物理设备一起提供给购买者（所有者）的文件。）
一个名为Simulation-mfg.sh的示例脚本，用于在设备制造商将在物理设备上运行的VM“设备”上运行启用SDO的步骤。这样，您可以在购买支持SDO的设备之前尝试使用Horizon实例进行SDO处理。
名为owner-boot-device的脚本，通过在启用了SDO的物理设备上运行的VM上启动相同的SDO引导过程，执行使用模拟VM“设备”的后半部分。

###使用SDO支持
执行以下步骤来试用Horizon SDO支持：

- [启动SDO所有者服务（只需要在第一时间完成）](https://github.com/open-horizon/SDO-support#start-services)
- [生成所有者密钥对](https://github.com/open-horizon/SDO-support#gen-keypair)
- [使用SDO初始化设备](https://github.com/open-horizon/SDO-support#init-device)
- [导入所有权凭证](https://github.com/open-horizon/SDO-support#import-voucher)
- [引导设备进行配置](https://github.com/open-horizon/SDO-support#boot-device)

####启动SDO所有者服务
SDO所有者服务会响应引导设备，并使管理员能够导入所有权凭证。这些所有者服务捆绑在一个容器中，通常由[All-in-1](https://github.com/open-horizon/devops/blob/master/mgmt-hub/README.md)开发人员管理中心或提供Horizon生产版本的供应商在安装Horizon管理中心时启动。

如果出于开发目的需要运行自己的SDO所有者服务实例，请参阅[启动SDO所有者服务自己的实例](https://github.com/open-horizon/SDO-support#start-services-developer)。

#####验证SDO所有者服务API端点
在继续执行其余的SDO流程之前，最好先确认您具有到达SDO所有者服务端点所需的正确信息。在Horizo​​n“管理员”主机上，运行这些简单的SDO API，以验证Docker容器中的服务是否可访问并正确响应。
（Horizon管理员主机是指已安装Horizo​​n-cli软件包，提供hzn命令，环境变量HZN_EXCHANGE_URL，HZN_SDO_SVC_URL，HZN_ORG_ID和HZN_EXCHANGE_USER_AUTH正确设置的主机）。

1.导出这些环境变量以用于后续步骤。 请查看管理中心安装程序以获取确切值：
```bash
export HZN_ORG_ID=<exchange-org>
export HZN_EXCHANGE_USER_AUTH=<user>:<password>
export HZN_SDO_SVC_URL=<protocol>://<sdo-owner-svc-host>:<ocs-api-port>/<ocs-api-path>
export SDO_RV_URL=http://sdo-sbx.trustedservices.intel.com:80
```
> 注意：Open Horizon SDO支持代码包括一个内置的集合服务器，可以在开发或空白环境中使用。 请参阅在开发/测试期间运行SDO支持。

2.查询OCS API版本：
```bash
curl -sS $HZN_SDO_SVC_URL/version && echo
```
3.查询已导入的所有权凭证（最初它将是一个空列表）：
```bash
# either use curl directly
curl -sS -w "%{http_code}" -u "$HZN_ORG_ID/$HZN_EXCHANGE_USER_AUTH" $HZN_SDO_SVC_URL/vouchers | jq
# or use the hzn command, if you have the horizon-cli package installed
hzn voucher list
```
4.“ Ping”集合服务器：
```bash
curl -sS -w "%{http_code}" -H Content-Type:application/json -X POST $SDO_RV_URL/mp/113/msg/20 | jq
```

####生成所有者密钥对
对于SDO的生产使用，您需要创建3个密钥对并将其导入所有者服务容器。 这些密钥对使您能够安全地接管支持SDO的设备制造商的SDO所有权凭证的所有权，并安全地配置启动SDO设备。 使用提供的generate-key-pair.sh脚本可以轻松创建必要的密钥对。 
（如果仅在开发/测试环境中试用SDO，则可以使用内置的示例密钥对，并跳过本节。以后始终可以添加自己的密钥对。）

> 注意：您只需执行一次本节中的步骤。 创建和导入密钥可以与所有设备一起使用。

1.转到要保存生成的密钥的目录，然后下载`generate-key-pair.sh`。
```bash
curl -sSLO https://github.com/open-horizon/SDO-support/releases/download/v1.8/generate-key-pair.sh
chmod +x generate-key-pair.sh
```
2.运行`generate-key-pair.sh`脚本。 系统将提示您回答一些问题，以便为您的私钥生成证书。 
（可以通过设置环境变量来避免出现提示。有关详细信息，
请运行`./generate-key-pair.sh -h。`）支持Ubuntu和macOS。
```bash
./generate-key-pair.sh
```
3.`generate-key-pair.sh`创建两个文件：
- `owner-keys.tar.gz`：包含3个私钥和关联证书的tar文件。 此文件将在下一步中导入到所有者服务容器中。
- `owner-public-key.pem`：相应的客户/所有者公共密钥（全部在单个文件中）。 设备制造商使用它来将凭证安全地扩展给所有者。 每当运行Simulation-mfg.sh时，将此文件作为参数传递，并将此公钥提供给为您生产启用SDO的设备的每个设备制造商。
4.在管理主机上，将`owner-keys.tar.gz`导入SDO所有者服务：
```bash
curl -sS -w "%{http_code}" -u "$HZN_ORG_ID/$HZN_EXCHANGE_USER_AUTH" -X POST -H Content-Type:application/octet-stream --data-binary @owner-keys.tar.gz $HZN_SDO_SVC_URL/keys && echo
```

####使用SDO初始化测试VM设备
如果您已经具有启用了SDO的物理设备，则可以改用该设备，并跳过本节。

名为`simulator-mfg.sh`的示例脚本模拟了启用SDO的设备制造商的步骤：使用SDO和凭据初始化“设备”，创建所有权凭证，并将其扩展给您。 在要初始化的VM设备上执行以下步骤（这些步骤适用于Ubuntu 18.04）：
```bash
mkdir -p $HOME/sdo && cd $HOME/sdo
curl -sSLO https://github.com/open-horizon/SDO-support/releases/download/v1.8/simulate-mfg.sh
chmod +x simulate-mfg.sh
export SDO_RV_URL=http://sdo-sbx.trustedservices.intel.com:80
export SDO_SAMPLE_MFG_KEEP_SVCS=true   # makes it faster if you run multiple tests
./simulate-mfg.sh
```
这将在文件`/var/sdo/voucher.json`中创建所有权凭证。

####导入所有权凭证
上一步中为设备创建的所有权凭证需要导入SDO所有者服务。在Horizon管理员主机上：

1.购买支持SDO的物理设备时，会从制造商处收到所有权凭证。 对于已配置为模拟启用SDO的设备的VM设备，类似的步骤是将文件`/var/sdo/voucher.json`从您的VM设备复制到此处。
2.导入所有权凭证，指定应使用策略初始化此设备以运行helloworld示例边缘服务：
```bash
hzn voucher import voucher.json -e helloworld
```

####引导设备进行配置
当启用了SDO的设备启动时，它将启动SDO进程。 它要做的第一件事是联系集合服务器，该服务器将其重定向到Horizon实例中的SDO所有者服务。 要在您的VM设备中模拟此过程，请执行以下步骤：
1.回到您的VM设备上，运行`owner-boot-device`脚本：
```bash
/usr/sdo/bin/owner-boot-device ibm.helloworld
```
2.现在，您的VM设备已配置为Horizon边缘节点，并已注册到Horizon管理中心，以运行helloworld示例边缘服务。 查看边缘服务的日志：
```bash
hzn service log -f ibm.helloworld
```
既然SDO已配置了边缘设备，则将在此设备上自动将其禁用，以便在重新启动设备时SDO将不会运行。 
（SDO的唯一目的是配置全新的设备。）要对边缘设备进行日常维护和配置，请使用`hzn`命令。 例如，您可以通过以下两种方式之一更新节点策略，以使其他边缘服务部署到该策略：
- 在此边缘设备上：`hzn policy update ...`
- 集中在管理主机上：`hzn exchange node updatepolicy <node-id> ...`
（使用-h标志可查看每个命令的帮助。）

#####故障排除
- 如果边缘设备未在Horizon管理中心中注册，请在`/var/sdo/agent-install.log`中查找错误消息。
- 如果边缘设备已注册，但没有边缘服务启动，请运行以下命令进行调试：
```bash
hzn eventlog list
hzn deploycheck all -b policy-ibm.helloworld_1.0.0 -o <exchange-org> -u iamapikey:<api-key>
```

###仅限开发人员
这些步骤仅需要由该项目的开发人员执行。

####为Open Horizon构建SDO所有者服务
1.从[Intel SDO版本1.8](https://github.com/secure-device-onboard/release/releases/tag/v1.8.0)中下载以下tar文件到目录`sdo/`并对其进行打包：
```bash
mkdir -p sdo && cd sdo
curl --progress-bar -LO https://github.com/secure-device-onboard/release/releases/download/v1.8.0/iot-platform-sdk-v1.8.0.tar.gz
tar -zxf iot-platform-sdk-v1.8.0.tar.gz
curl --progress-bar -LO https://github.com/secure-device-onboard/release/releases/download/v1.8.0/rendezvous-service-v1.8.0.tar.gz
tar -zxf rendezvous-service-v1.8.0.tar.gz
curl --progress-bar -LO https://github.com/secure-device-onboard/release/releases/download/v1.8.0/pri-v1.8.0.tar.gz
tar -zxf pri-v1.8.0.tar.gz
curl --progress-bar -LO https://github.com/secure-device-onboard/release/releases/download/v1.8.0/NOTICES.tar.gz
tar -zxf NOTICES.tar.gz
cd ..
```
2.构建将运行Open Horizon所需的所有SDO服务的Docker容器：
```bash
# first update the VERSION variable value in Makefile, then:
make sdo-owner-services
```
3.亲自测试服务后，使用`testing`标签将其推送到docker hub，以便开发团队中的其他人可以对其进行测试：
```bash
make push-sdo-owner-services
```
4.开发团队验证服务后，将其发布到docker hub，作为带有最新标签的`latest`补丁程序发布：
```bash
make publish-sdo-owner-services
```
5.在经过充分测试的发行边界上（通常在版本的第二个数字更改时），使用被认为稳定的标签将其发布到docker hub：
```bash
make promote-sdo-owner-services
```

####生成模拟设备制造商文件
[可在示例SDO制造商脚本和Docker映像的“仅限开发人员”部分中找到这些步骤。](https://github.com/open-horizon/SDO-support/blob/master/sample-mfg/README.md#developers-only)

####启动您自己的SDO所有者服务实例
SDO所有者服务会响应引导设备，并使管理员能够导入所有权凭证。 提供以下4种服务：

- RV：集合服务器的开发版本，这是每个启用了SDO的启动设备都联系的初始服务。 RV将设备重定向到与正确的Horizon Management Hub关联的OPS。
- OPS：所有者协议服务与设备通信，并安全下载设备配置脚本和文件。
- OCS：Owner Companion Service管理包含设备配置信息的数据库文件。
- OCS-API：一种REST API，支持导入和查询所有权凭证。

SDO所有者服务打包为单个docker容器，可以在对Horizon Management Hub具有网络访问权限且SDO设备可以通过网络访问的任何服务器上运行。

1.获取用于启动容器的`run-sdo-owner-services.sh`：
```bash
mkdir $HOME/sdo; cd $HOME/sdo
curl -sSLO https://raw.githubusercontent.com/open-horizon/SDO-support/v1.8/docker/run-sdo-owner-services.sh
chmod +x run-sdo-owner-services.sh
```
2.运行`./run-sdo-owner-services.sh -h`查看用法，并设置所有必需的环境变量。 例如：
```bash
export HZN_EXCHANGE_URL=https://<cluster-url>/edge-exchange/v1
export HZN_FSS_CSSURL=https://<cluster-url>/edge-css
export HZN_ORG_ID=<exchange-org>
export HZN_EXCHANGE_USER_AUTH=iamapikey:<api-key>
```
3.为所有者服务容器内的主密钥库选择或生成一个密码，然后将其分配给SDO_KEY_PWD。 例如：
```bash
export SDO_KEY_PWD=123456
```
>注意：将密码保存在安全，持久的位置。 每次重新启动时，都需要将此相同的密码传递到所有者服务容器中。 
>如果您忘记了现有主密钥库的密码，则必须删除容器，删除卷，然后返回到[启动SDO所有者服务](https://github.com/open-horizon/SDO-support#start-services)。

4.作为安装Horizon管理中心的一部分，您应该已经运行[edgeNodeFiles.sh](https://github.com/open-horizon/anax/blob/master/agent-install/edgeNodeFiles.sh)，它创建了一个包含`agent-install.crt`的tar文件。 使用它来导出此环境变量：
```bash
export HZN_MGMT_HUB_CERT=$(cat agent-install.crt | base64)
```
5.启动SDO所有者服务docker容器并查看日志：
```bash
./run-sdo-owner-services.sh
docker logs -f sdo-owner-services
```
6.[执行验证SDO所有者服务API端点](https://github.com/open-horizon/SDO-support#verify-services)中的用户验证步骤。
7.执行以下其他开发者验证步骤： 
```bash
guid=$(jq -r .oh.g voucher.json)   # if you have a current voucher
guid='ox0fh+4kSJC6vbpCTpKbLg=='  # if you do NOT have a current voucher
curl -sS -w "%{http_code}" -H Content-Type:application/json -d '{"g2":"'$guid'","n5":"qoVoYKjUn7d6g3KaBrFXfQ==","pe":1,"kx":"ECDH","cs":"AES128/CTR/HMAC-SHA256","eA":[13,0,""]}' -X POST http://$SDO_OWNER_SVC_HOST:$SDO_OPS_EXTERNAL_PORT/mp/113/msg/40 | jq
# an http code of 200 or 400 means it can successfully communicate with OPS
curl -sS -w "%{http_code}" -H Content-Type:application/json -d '{"g2":"'$guid'","eA":[13,0,""]}' -X POST $SDO_RV_URL/mp/113/msg/30 | jq
# an http code of 200 or 500 means it can successfully communicate with RV
```
####在开发/测试期间运行SDO支持
按照[使用SDO支持](https://github.com/open-horizon/SDO-support#use-sdo)中的说明进行操作时，设置以下环境变量以与您或团队中其他人正在开发的最新文件和docker映像一起使用：

- 在[“启动SDO所有者服务”集](https://github.com/open-horizon/SDO-support#start-services)中： 
```bash
# to use the most recently committed version of agent-install.sh:
export AGENT_INSTALL_URL=https://raw.githubusercontent.com/open-horizon/anax/master/agent-install/agent-install.sh
# if the hostname of this host is not resolvable by the device, provide the IP address to RV instead
export SDO_OWNER_SVC_HOST="1.2.3.4"
# use the built-in rendezvous server instead of Intel's global RV
export SDO_RV_URL=http://<sdo-owner-svc-host>:8040
# when running agent-install.sh on the edge device, it should get pkgs from CSS
export SDO_GET_PKGS_FROM=css:
# set your own password for the master keystore
export SDO_KEY_PWD=<pw>
# when curling run-sdo-owner-services.sh use the master branch instead of the 1.8 tag
```
- 在[设置了SDO的设备初始化](https://github.com/open-horizon/SDO-support#init-device)中：
```bash
# set SDO_SUPPORT_REPO 1 of these 2 ways:
export SDO_SUPPORT_REPO=https://raw.githubusercontent.com/open-horizon/SDO-support/master   # using owner-boot-device from the most recent committed upstream
export SDO_SUPPORT_REPO=https://raw.githubusercontent.com/<my-github-id>/SDO-support/<my-branch>   # using owner-boot-device from the branch you are working on
# use the built-in rendezvous server instead of Intel's global RV
export SDO_RV_URL=http://<sdo-owner-svc-host>:8040
# set SDO_MFG_IMAGE_TAG 1 of these 2 ways:
export SDO_MFG_IMAGE_TAG=testing   # using the most recent development docker image from the team
export SDO_MFG_IMAGE_TAG=1.2.3   # using the docker image you are still working on
# this will speed repetitive testing, because it will leave the mfg containers running if they haven't changed
export SDO_SAMPLE_MFG_KEEP_SVCS=true
# when curling simulate-mfg.sh use the master branch instead of the 1.8 tag
```
- 在[“导入所有权凭证”集](https://github.com/open-horizon/SDO-support#import-voucher)中：（到目前为止没有什么特别的）
- 在[启动设备进行配置中](https://github.com/open-horizon/SDO-support#boot-device)设置：（到目前为止没有什么特别的）
