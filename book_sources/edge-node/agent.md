### Horizon Agent API
本文档包含在边缘节点上运行的Horizon Agent的Horizon REST API。 API的输出为JSON紧凑格式。为了获得更好的视图，可以在Web浏览器中使用JSONView扩展名，也可以从命令行界面使用`jq`命令。例如：

```bash
curl -s http://<ip>/status | jq '.'
```

#### **API:** GET/status
获取节点上的连接性和配置状态。输出包括代理配置的状态和节点的连接性。

**参数:**

none

**返回:**

code:
* 200 -- success

body:

| name | subfield | type | description |
| ---- | ---- |----| ---------------- |
| configuration||json| the configuration data.  |
| |exchange_api| string | the url for the exchange being used by the Horizon agent. |
| |exchange_version | string | the current version of the exchange being used. |
| |required_minimum_exchange_version | string | the required minimum version for the exchange. |
| |preferred_exchange_version | string | the preferred version for the exchange in order to use all the horizon functions. |
| |mms_api| string | the url for the model management system. |
| |architecture | string | the hardware architecture of the node as returned from the Go language API runtime.GOARCH. |
| |horizon_version | string | The current version of the horiozn running on this node. |
| connectivity || json | whether or not the node has network connectivity with some remote sites. |

**例子:**
```bash
curl -s http://localhost:8510/status | jq '.'
{
  "configuration": {
    "exchange_api": "http://exchange-api:8080/v1/",
    "exchange_version": "2.15.1",
    "required_minimum_exchange_version": "2.15.1",
    "preferred_exchange_version": "2.15.1",
    "mms_api": "https://css-api:9443",
    "architecture": "amd64",
    "horizon_version": "2.24.5"
  },
  "liveHealth": null
}
```

#### **API:** GET/status/workers
获取当前Horizo​​n Agent工作人员状态和状态事务日志。
**参数:**

none

**返回:**

code:
* 200 -- success

body:

| name | subfield | type | description |
| ---- | ---- |----| ---------------- |
| workers  | | json | the current status of each worker and its subworkers. |
| | name | string | the name of the worker. |
| | status | string | the status of the worker. The valid values are: added, started, initialized, initialization failed, terminating, t erminated. |
| | subworker_status | json | the name and the status of the subworkers that are created by this worker. |
| worker_status_log | | string array |  the history of the worker status changes. |

**例子:**
```bash
curl -s  http://localhost:8510/status/workers |jq
{
  "workers": {
    "AgBot": {
      "name": "AgBot",
      "status": "terminated",
      "subworker_status": {}
    },
    "Agreement": {
      "name": "Agreement",
      "status": "initialized",
      "subworker_status": {}
    },
    "Container": {
      "name": "Container",
      "status": "initialized",
      "subworker_status": {}
    },
    "ExchangeChanges": {
      "name": "ExchangeChanges",
      "status": "initialized",
      "subworker_status": {}
    },
    "ExchangeMessages": {
      "name": "ExchangeMessages",
      "status": "initialized",
      "subworker_status": {}
    },
    "Governance": {
      "name": "Governance",
      "status": "initialized",
      "subworker_status": {
        "ContainerGovernor": "started",
        "MicroserviceGovernor": "started",
        "SurfaceExchErrors": "started"
      }
    },
    "ImageFetch": {
      "name": "ImageFetch",
      "status": "initialized",
      "subworker_status": {}
    },
    "Kube": {
      "name": "Kube",
      "status": "initialized",
      "subworker_status": {}
    },
    "Resource": {
      "name": "Resource",
      "status": "initialized",
      "subworker_status": {}
    }
  },
  "worker_status_log": [
    "2020-03-27 19:06:07 Worker AgBot: started.",
    "2020-03-27 19:06:07 Worker AgBot: initialization failed.",
    "2020-03-27 19:06:07 Worker Agreement: started.",
    "2020-03-27 19:06:07 Worker Governance: started.",
    "2020-03-27 19:06:07 Worker ExchangeMessages: started.",
    "2020-03-27 19:06:07 Worker ExchangeMessages: initialized.",
    "2020-03-27 19:06:07 Worker Container: started.",
    "2020-03-27 19:06:07 Worker AgBot: terminated.",
    "2020-03-27 19:06:07 Worker ImageFetch: started.",
    "2020-03-27 19:06:07 Worker ImageFetch: initialized.",
    "2020-03-27 19:06:07 Worker Kube: started.",
    "2020-03-27 19:06:07 Worker Kube: initialized.",
    "2020-03-27 19:06:07 Worker Resource: started.",
    "2020-03-27 19:06:07 Worker Resource: initialized.",
    "2020-03-27 19:06:07 Worker ExchangeChanges: started.",
    "2020-03-27 19:06:07 Worker ExchangeChanges: initialized.",
    "2020-03-27 19:06:07 Worker Container: initialized.",
    "2020-03-27 19:06:12 Worker Agreement: initialized.",
    "2020-03-27 19:19:32 Worker Governance: subworker SurfaceExchErrors added.",
    "2020-03-27 19:19:32 Worker Governance: subworker ContainerGovernor added.",
    "2020-03-27 19:19:32 Worker Governance: subworker MicroserviceGovernor added.",
    "2020-03-27 19:19:32 Worker Governance: subworker SurfaceExchErrors started.",
    "2020-03-27 19:19:32 Worker Governance: subworker ContainerGovernor started.",
    "2020-03-27 19:19:32 Worker Governance: subworker MicroserviceGovernor started.",
    "2020-03-27 19:19:32 Worker Governance: initialized."
  ]
}

```

具体各个API的信息请参阅 https://github.com/open-horizon/anax/blob/master/docs/api.md