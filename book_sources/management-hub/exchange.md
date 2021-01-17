##Horizon Data Exchange Server和REST API

数据交换API为交换Web UI（未来），边缘节点和agreement Bots提供API服务。

交换服务还为BH在分散的P2P工具尚未充分扩展的领域提供了一些关键服务。 一旦去中心化工具足够，它们将在交易所中替换这些服务。

###本地开发的准备
- [安装scala](http://www.scala-lang.org/download/install.html)

- [安装sbt](https://www.scala-sbt.org/1.x/docs/Setup.html)

- （可选）如果要从scalatra.org获取示例代码，请安装conscript和giter8
  
- 在本地安装postgresql（除非您使用的是远程实例）。在Mac OS X上的安装说明：
  
  - 安装：`brew install postgresql`
  - 注意：在docker容器中运行/测试Exchange svr时，它无法访问本地主机上的postgres实例，因此将其配置为还侦听您的本地IP：
    
    - 将此设置为您的IP：export MY_IP=<my-ip>
      
    - `echo "host all all $MY_IP/32 trust" >> /usr/local/var/postgres/pg_hba.conf`
      
    - `sed -i -e "s/#listen_addresses = 'localhost'/listen_addresses = '$MY_IP'/" /usr/local/var/postgres/postgresql.conf`
      
    - `brew services start postgresql ` 或者如果它已经在运行brew服务，则重新启动postgresql `brew services restart postgresql`
      
   - 或者，如果您的测试计算机在专用子网中：
   
     - 信任子网上的所有客户端：`echo'host all all 192.168.1.0/24 trust'>>`
       `/usr/local/var/postgres/pg_hba.conf`
     - 监听所有接口：`sed -i -e "s /＃listen_addresses ='localhost'/ listen_addresses ='*'/"` 
       `/usr/local/var/postgres/postgresql.conf`
     - `brew service start postgresql` 或者如果它已经在运行`brew services restart postgresql`
   - 测试：`psql" host = $ MY_IP dbname = postgres user = <myuser> password =``"`
 -  在您的开发系统上添加一个配置文件`/etc/horizon/exchange/config.json`，该文件至少包含以下内容（自动化测试需要此内容。默认值和配置变量的完整列表位于`src/main/resources/config.json`）：
   ```bash
   {
     "api": {
       "db": {
         "jdbcUrl": "jdbc:postgresql://localhost/postgres",    // my local postgres db
         "user": "myuser",
         "password": ""
       },
       "logging": {
         "level": "DEBUG"
       },
       "root": {
         "password": "myrootpw"
       }
     }
   }
  ```
- 如果要运行`FrontEndSuite`测试类，则c`onfig.json`还应在`root`目录下的"email"后面直接包含`"frontEndHeader"："issuer"`。
- 在您的Shell环境中设置相同的交换根密码，例如：
`export EXCHANGE_ROOTPW=myrootpw`
- 如果尚未完成，请创建TLS私钥和证书：
`export EXCHANGE_KEY_PW=<pass-phrase>
 make gen-key
 `
- 否则，请从创建它们的人那里获取文件`exchangecert.pem`，`keypassword`和`keystore`，并将它们放在 `./keys/etc中`。

### 在本地沙箱中构建和运行
- `sbt`
- `~reStart`
- 服务器启动后，要查看swagger输出，请浏览`：http：//localhost：8080/v1/swagger`
- 要尝试简单的rest方curl：`curl -X GET "http：//localhost：8080/v1/admin /version"`。 您应该获得交换版本号作为响应。
- 可以运行便捷脚本`src/test/bash/primedb.sh`来为DB填充一些交换资源，以用于手动测试：
```bash
export EXCHANGE_USER=<my-user-in-IBM-org>
export EXCHANGE_PW=<my-pw-in-IBM-org>
src/test/bash/primedb.sh
```
- `primedb.sh`只会创建尚不存在的内容，因此可以再次运行它以恢复已删除的某些资源。
- 要针对现有ICP群集在本地测试交换，请执行以下操作：
```bash
export ICP_EXTERNAL_MGMT_INGRESS=<icp-external-host>:8443
```

### 使用Sbt的技巧

在`sbt`子命令提示符下：

- 获取任务列表：`task -V`
- 启动您的应用程序，使其在代码更改时重新启动：`〜reStart`
- 清除所有构建文件（如果需要重置增量构建）：`clean`

### 在本地沙箱中运行自动测试

- （可选）包括对IBM Agbot ACL的测试：`export EXCHANGE_AGBOTAUTH = myibmagbot：abcdef`
- （可选）包括IBM IAM平台密钥认证的测试：
```bash
导出EXCHANGE_IAM_KEY = myiamplatformkey
导出EXCHANGE_IAM_EMAIL=myaccountemail@something.com
导出EXCHANGE_IAM_ACCOUNT = myibmcloudaccountid
```
- 在第二个外壳中运行自动化测试（交换服务器仍在第一个外壳中运行）：`sbt test`
- 仅运行其中一个自动化测试套件（交换服务器仍在运行）：`sbt"testOnly **。AgbotsSuite"`
- 运行性能测试：`src/test/bash/scale/test.sh`或`src/test/bash/scale/wrapper.sh 8`
- 确保在运行`AgbotsSuite`测试类之前运行`primedb.sh`以运行所有测试。

###代码覆盖率报告

默认情况下，项目中的代码覆盖率处于禁用状态。 sbt命令`sbt coverage`可以打开/关闭覆盖率检查。 要创建一份覆盖率报告：
  
- 执行`sbt coverage`以启用覆盖。
- 运行所有测试，请参见上面的部分（“在本地沙箱中运行自动测试”）。
- 创建报告运行命令`sbt coverageReport`。
- 终端将显示报告的撰写位置，并提供高级百分比摘要。

### Linting

项目使用Scapegoat。 使用方法：
  
- 运行`sbt scapegoat`
- 终端将显示报告的编写位置，并提供发现的错误和警告的摘要。

### 构建和运行Docker容器

- 更新src / main / resources.version.txt中的版本
- **此步骤尚不可用，暂时使用以下项目符号中的2个步骤
  - 要编译本地代码，构建交换容器并运行它，请运行：`make`
- 或者您可以按单独的步骤进行构建和运行：
    - 编译本地代码并构建交换容器：`make .docker-exec`
    - 在本地运行容器：`make start-docker-exec`
- 在本地手动测试容器：`curl -sS -w％{http_code} http：//localhost：8080/v1/admin/version`
    - 注意：如果该容器仅侦听Unix域套接字或127.0.0.1，则该容器无法访问在主机上本地运行的postgres数据库。请参阅上面的“前提条件”部分。
- 通过https手动测试本地容器：
    - 如果尚未安装，请设置EXCHANGE_KEY_PW并运行`make gen-key`
    - 在`/etc/hosts`中添加`edge-fab-exchange`作为`localhost'的别名
    -`src/test/bash/https.sh获得服务`
- 运行自动化测试：`sbt test`
- 检查容器中的swagger信息：`http：//localhost：8080/v1/swagger`
- 可以通过`docker logs -f exchange-api`查看exchange svr的日志输出，或者也可以转到`/var/log/syslog`，具体取决于docker和syslog配置。
- 在这一点上，您可能想进行`make clean`操作以停止本地docker容器，以使其停止在8080端口上侦听，否则当您返回在沙箱中运行新代码时，您会非常困惑，而您的测试却没有似乎正在执行它。

#### 关于`config/exchange-api.tmpl`的注意事项

- `config/exchange-api.tmpl`是一个应用程序配置模板，类似于`/etc/horizon/exchange/config.json`。模板文件本身是构建Docker映像所必需的，但内容不是必需的。建议在构建Docker映像时默认内容保持原样。
- 模板的内容布局与`/etc/horizon/exchange/config.json`的布局完全匹配，并且config.json的内容可以直接复制并粘贴到模板中。创建Docker容器时，这会将默认Exchange配置设置为config.json中定义的硬编码规范。
- 另外，模板也可以使用替代变量（`config/exchange-api.tmpl`的默认内容）代替使用硬编码的值。在创建容器时，实用程序`envsubt`将用Docker，Kubernetes，Openshift等传递给运行容器的任何相应环境变量替换值。
    - `config / exchange-api.tmpl`：
        - "jdbcUrl"："$ EXCHANGE_DB_URL"
    - Kubernetes config-map（环境变量在创建时传递给容器）：
        - "$EXCHANGE_DB_URL = 192.168.0.123"
    - 运行容器中的默认`/etc/horizon/exchange/config.json`：
        - "jdbcUrl"："192.168.0.123"
- 可以在模板中混合匹配硬编码值和替换值。
- ***警告***`envsubst`会尝试替换任何包含`$`字符的值（例如哈希密码）。为了防止这种情况，要么通过环境变量`$ENVSUBST_CONFIG`带有垃圾值（这将有效地禁用`envsubst`），要么通过包含确切替代变量`envsubst`的值进行替换（`$ENVSUBST_CONFIG ='${ EXCHANGE_DB_URL} ${EXCHANGE_DB_USER} ${EXCHANGE_DB_PW} ${EXCHANGE_ROOT_PW} ...'`）和正常的环境变量一起实际进行值替换。
    - 默认情况下，`$ENVSUBST_CONFIG`设置为`$ENVSUBST_CONFIG ="`，这基本上是`envsubst`，使用默认的机会主义行为，并会在可能的情况下尝试进行任何/所有替换。
- 也可以在创建时使用绑定/卷挂载将`/etc/horizon/exchange/config.json`直接传递到容器。这优先于模板`config/exchange-api.tmpl`的内容。直接传递的config.json仍然受`envsubt`实用程序的约束，并且上述警告仍然适用。

#### 关于Docker映像构建过程的注意事项

- 参见https://www.codemunity.io/tutorials/dockerising-akka-http/
- 使用sbt-native-packager插件。请参阅上面的URL，了解要添加到与sbt相关的文件中的内容
- 构建docker映像：`sbt docker：publishLocal`
- 手动生成并运行交换可执行文件
  - `make runexecutable`
- 要查看创建的dockerfile：
  - `sbt docker：stage`
  - `cat target / docker / stage / Dockerfile`
  
###使用Anax测试交换

-如果要在另一台计算机上使用anax进行测试，则仅将版本标记的交换映像推送到docker hub，以便其他计算机可以使用：`make docker-push-version-only`
-第一次：在ubuntu机器上，克隆anax存储库并定义以下e2edev脚本：
```bash
mkdir -p ~/src/github.com/open-horizon && cd ~/src/github.com/open-horizon && git clone git@github.com:open-horizon/anax.git
# See: https://github.com/open-horizon/anax/blob/master/test/README.md
if [[ -z "$1" ]]; then
    echo "Usage: e2edev <exchange-version>"
    return
fi
set -e
sudo systemctl stop horizon.service   # this is only needed if you normally use this machine as a horizon edge node
cd ~/src/github.com/open-horizon/anax
git pull
make clean
make
make fss   # this might not be needed
cd test
make
make test TEST_VARS="NOLOOP=1 TEST_PATTERNS=sall" DOCKER_EXCH_TAG=$1
make stop
make test TEST_VARS="NOLOOP=1" DOCKER_EXCH_TAG=$1
echo 'Now run: cd $HOME/src/github.com/open-horizon/anax/test && make realclean && sudo systemctl start horizon.service && cd -'
set +e
```
- 现在运行测试（大约需要10分钟）：
```bash
- Now run the test (this will take about 10 minutes):
```

###将容器部署到暂存或生产

- 将容器推送到docker hub注册表：`make docker-push-only`
- 将新容器部署到登台或生产Docker主机
    - 确保不需要更改/etc/horizon/exchange/config.json文件
- 测试新容器：`curl -sS -w％{http_code} https：// <exchange-host> / v1 / admin / version`

###为分支机构构建容器

要使用针对git分支的代码构建交换容器，请执行以下操作：
- 创建一个开发git分支A（经过测试，您将合并到分支B）。其中A和B可以是任何分支名称。
- 通过sbt在本地测试分支A交换
- 当所有测试通过时，构建容器：`rm -f .docker-compile && make .docker-exec-run TARGET_BRANCH = B`
- 上面的命令将创建一个标记为：1.2.3-B的容器
- 测试交换容器
- 当所有测试通过时，将其推送到docker hub：`make docker-push-only TARGET_BRANCH = B`
- 上面的命令将推送标记为1.2.3-B和Latest-B的容器
- 创建PR将您的开发分支A合并到canonical分支B

###交换根用户

####将哈希密码放入config.json

交换根用户密码在配置文件（`/etc/horizon/exchange/config.json`）中设置。但是密码不需要是明文。您可以使用以下方式对密码进行哈希处理：
```bash
curl -sS -X POST -H"Authorization:Basic $ HZN_ORG_ID / $ HZN_EXCHANGE_USER_AUTH" -H"Content-Type：application / json” -d'{“password”：“ PUT-PW-HERE”}'$ HZN_EXCHANGE_URL/admin/hashpw | q
```

然后将该哈希值放在`api.root.password`字段的`/etc/horizon/exchange/config.json`中。

####禁用root用户

如果要减少交换的攻击面，则可以禁用交换根用户，因为仅在特殊情况下才需要。建议您在禁用root之前执行以下操作：
- 在IBM组织中创建本地交换用户。 （如果您想在某些时候使用`https：//raw.githubusercontent.com/open-horizon/examples/master/tools/exchangePublishScript.sh`更新示例服务，模式和策略，可以使用此方法。）：
  
  ```bash
  hzn exchange user create -u "root/root：PUT-ROOT-PW-HERE" -o IBM -A PUT-USER-HERE PUT-PW-HERE PUT-EMAIL-HERE
  ```

- 授予1个IBM Cloud用户`admin`特权：
  ```bash
  hzn exchange usersetadmin -u "root/root：PUT-ROOT-PW-HERE" -o PUT-IBM-CLOUD-ORG-HERE PUT-USER-HERE true
  ```

现在，您可以通过在`/etc/horizon/exchange/config.json`中将`api.root.enabled`设置为false来禁用root。

###将来版本中可能要做的待办事项

- 粒度（按组织）服务ACL支持：
    - 添加“浏览”的访问类型，该类型将允许查看以下服务字段：
      - 标签，说明，公开，文档，URL，版本，拱
    - 添加带有字段的ACL表和资源：
      - 组织
      - 资源（例如服务）
      - resourceList（供将来使用）
      - 请求者（组织/用户名）
      - 访问
    - 添加GET，PUT，POST以管理这些ACL
    - 在GET服务上添加检查以使用这些ACL
    - （稍后）考虑添加一个GET，该GET返回orgType = IBM的org的所有服务
- 添加rest 方法以删除用户的过时设备（carl 请求）
- 增加了更改节点所有者的能力
- 为节点注册服务添加补丁功能
- 考虑：
    - 检测模式是否包含2个依赖于相同排他MS的服务
    - 检测模式是否已使用具有userInput w/o默认值的服务更新，并发出警告
    - 考虑将所有创建内容更改为POST，并将（通过 put/patch）返回码更新为200