## Horizon Agreement Bot API
本文档包含用于运行协议Bot的Horizon系统的Horizon JSON API。 API的输出为JSON紧凑格式。 为了获得更好的视图，您可以在Web浏览器中使用JSONView扩展名，或者从命令行界面使用`jq`命令。 例如：
```bash
curl -s http://<ip>/agreement | jq '.'
```

### 1. Horizon Agreement Bot Remote APIs
可以从远程节点运行以下API。 它们是安全的API，这意味着您需要使用HTTPS和Agreenent Bot提供的CA证书文件来运行。 您还需要从Exchange提供用户名和密码（或API密钥）以进行验证和身份验证。 例如：
```bash
curl -sLX GET -w％{http_code} --cacert <cert_file_name> -u myord / myusername：mypassword --data @-https://123.456.78.9:8083/deploycheck/deploycompatible
```
#### 1.1部署兼容性检查
##### **API:** GET  /deploycheck/deploycompatible
---

This API does compatibility check for the given business policy (or a pattern), service definition, node policy and node user input. It does both policy compatibility check and user input compatibility check. If the result is compatible, it means that, when deployed, the node will form an agreement with the agbot and the service will be running on the node.

**参数:**

query paramters:

| name | type | description |
| ---- | ---- | ---------------- |
| checkAll | boolean | return the compatibility check result for all the service versions referenced in the business policy or pattern. |
| long | boolean | show the input which was used to come up with the result. |

body:

| name | type | description |
| ---- | ---- | ---------------- |
| node_id | string | the exchange id of the node. Mutually exclusive with node_policy and node_user_input.|
| node_arch | string | (optional) the architecture of the node. |
| node_policy | json | the node policy that will be put in the exchange. Mutually exclusive with node_id. Please refer to [node policy sample](https://github.com/open-horizon/anax/blob/master/cli/samples/node_policy_input.json) for the format. |
| node_user_input | json | the user input that will be put in the exchange for the services. Mutually exclusive with node_id. Please refer to [node user input sample](https://github.com/open-horizon/anax/blob/master/cli/samples/user_input.json) for the format. |
| business_policy_id   | string | the exchange id of the business policy. Mutually exclusive with business_policy. Mutually exclusive with pattern_id and pattern.|
| business_policy | json | the defintion of the business policy that will be put in the exchange. Mutually exclusive with business_policy_id. Mutually exclusive with pattern_id and pattern. Please refer to [business policy sample](https://github.com/open-horizon/anax/blob/master/cli/samples/business_policy.json) for the format. |
| pattern_id | string | the exchange id of the pattern. Mutually exclusive with pattern. Mutually exclusive with business_policy_id and business_policy. |
| pattern | json | the pattern that will be put in the exchange. Mutually exclusive with pattern_id. Mutually exclusive with business_policy_id and business_policy. Please refer to [pattern sample](https://github.com/open-horizon/anax/blob/master/cli/samples/pattern.json) for the format. |
| service_policy   | json | (optional) the service policy that will be put in the exchange for the top level service referenced in the business policy. If omitted, the service policy will be retrieved from the exchange. The service policy has the same format as the node policy. Please refer to [node policy sample](https://github.com/open-horizon/anax/blob/master/cli/samples/node_policy_input.json) for the format. |
| service | json array | (optional) an array of the top level services that will be put in the exchange. They are refrenced in the business policy or pattern. If omitted, the services will be retrieved from the exchange. Please refer to [service sample](https://github.com/open-horizon/anax/blob/master/cli/samples/service.json) for the format. |

**返回:**
code: 
* 200 -- success

body:

| name | type | description |
| ---- | ---- | ---------------- |
| compatible | bool | the deployment resources are compatible or not. |
| reason | map | the key is the exchange id for a service and the value is the reason why this service is not compatible. It lists reasons for all the service versions referenced in the business policy (or pattern) if checkAll=1 is set in the url. |
| input | json | the input which is used to come up with the compatibility check result. It has the same structure as the paramter body above but with details filled by the code. For example, if a business policy id is given, the business policy will be retrieved from the exchange and set in the input field. The input is only shown when the API is called with long=1 in the url. |

**例子 :**
```bash
read -d '' comp_input <<EOF
{
  "node_id":  "userdev/an12345,
  "business_policy_id": "userdev/bp_location"
}
EOF

echo "$comp_input" | curl -sLX GET -w %{http_code} --cacert <cert_file_name> -u myord/myusername:mypassword --data @- https://123.456.78.9:8083/deploycheck/deploycompatible | jq '.'
{
  "compatible": true,
  "reason": {
    "e2edev@somecomp.com/bluehorizon.network-services-location_2.0.6_amd64": "Compatible",
  }
}
 

echo "$comp_input" | curl -sLX GET -w %{http_code} --cacert <cert_file_name> -u myord/myusername:mypassword --data @- https://123.456.78.9:8083/deploycheck/deploycompatible?checkAll=1 | jq '.'
{
  "compatible": true,
  "reason": {
    "e2edev@somecomp.com/bluehorizon.network-services-location_2.0.6_amd64": "Compatible",
    "e2edev@somecomp.com/bluehorizon.network-services-location_2.0.7_amd64": "Policy Incompatible: Compatibility Error: Properties do not satisfy node constraint."
  }
}


echo "$comp_input" | curl -sLX GET -w %{http_code} --cacert <cert_file_name> -u myord/myusername:mypassword --data @- https://123.456.78.9:8083/deploycheck/deploycompatible?long=1 | jq '.'
{
  "compatible": true,
  "reason": {
    "e2edev@somecomp.com/bluehorizon.network-services-location_2.0.6_amd64": "Compatible"
  },
  "input": {
    "node_id": "userdev/an12345",
    "node_arch": "amd64",
    "node_policy": {
      "properties": [...],
      "constraints": [...]
    },
    "node_user_input": [
      {
        "serviceOrgid": "e2edev@somecomp.com",
        "serviceUrl": "https://bluehorizon.network/services/locgps",
        "serviceArch": "amd64",
        "serviceVersionRange": "2.0.3",
        "inputs": [...]
      }
     ],
    "business_policy": {
      "owner": "userdev/userdevadmin",
      "label": "business policy for location",
      ...
    },
    "service": [
      {
        "org": "e2edev@somecomp.com",
        "owner": "e2edev@somecomp.com/e2edevadmin",
        "url": "https://bluehorizon.network/services/location",
        ...
      }
    ]
  }
}

```

```bash
# three different ways of getting definitions of the resource:
bp_location=$(</user/me/input_files/compcheck/business_pol_location.json)
node_ui=`cat /user/me/input_files/compcheck/node_ui.json`
read -d '' node_pol <<EOF
{
  "properties": [
    {
      "name": "purpose",
      "value": "network-testing"
    },
    {
      "name": "group",
      "value": "bluenode"
    }
  ],
  "constraints": [
    "iame2edev == true",
    "NOLOC == false ",
    "openhorizon.service.version != 2.0.6"
  ]
}
EOF

read -d '' comp_input <<EOF
{
  "node_policy":      $node_pol,
  "node_user_input":  $node_ui,
  "business_policy":  $bp_location
}
EOF

echo "$comp_input" | curl -sLX GET -w %{http_code} --cacert <cert_file_name> -u myord/myusername:mypassword --data @- https://123.456.78.9:8083/deploycheck/deploycompatible | jq '.'
{
  "compatible": true,
  "reason": {
    "e2edev@somecomp.com/bluehorizon.network-services-location_2.0.6_amd64": "Compatible",
  }
}
```

具体各个API的信息请参阅 https://github.com/open-horizon/anax/blob/master/docs/agreement_bot_api.md