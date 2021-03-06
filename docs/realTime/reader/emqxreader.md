# Emqx Reader
<!-- TOC -->

- [Emqx Reader](#emqx-reader)
    - [一、插件名称](#一插件名称)
    - [二、支持的数据源版本](#二支持的数据源版本)
    - [三、参数说明<br />](#三参数说明br-)
    - [四、配置示例](#四配置示例)

<!-- /TOC -->

<br/>

<a name="c6v6n"></a>
## 一、插件名称
名称：**emqxreader**<br />

<br/>

<a name="jVb3v"></a>
## 二、支持的数据源版本
**Emqx 4.0及以上**<br />
<a name="2lzA4"></a>

<br/>

## 三、参数说明<br />

- **broker**
  - 描述：连接URL信息
  - 必选：是
  - 字段类型：String
  - 默认值：无
<br />


- **topic**
  - 描述：订阅主题
  - 必选：是
  - 字段类型：String
  - 默认值：无
<br />


- **username**
  - 描述：认证用户名
  - 必选：否
  - 字段类型:String
  - 默认值：无
<br />


- **password**
  - 描述：认证密码
  - 必选：否
  - 字段类型：String
  - 默认值：无
<br />


- **isCleanSession**
  - 描述：是否清除session
    - false：MQTT服务器保存于客户端会话的的主题与确认位置；
    - true：MQTT服务器不保存于客户端会话的的主题与确认位置
  - 必选：否
  - 字段类型：boolean
  - 默认值：true
<br />


- **qos**
  - 描述：服务质量
    - 0：AT_MOST_ONCE，至多一次；
    - 1：AT_LEAST_ONCE，至少一次；
    - 2：EXACTLY_ONCE，精准一次；
  - 必选：否
  - 字段类型：int
  - 默认值：2
<br />


- **codec**
  - 描述：编码解码器类型，支持 json、plain
    - plain：将kafka获取到的消息字符串存储到一个key为message的map中，如：`{"message":"{\"key\": \"key\", \"message\": \"value\"}"}`
    - plain：将kafka获取到的消息字符串按照json格式进行解析
      - 若该字符串为json格式
        - 当其中含有message字段时，原样输出，如：`{"key": "key", "message": "value"}`
        - 当其中不包含message字段时，增加一个key为message，value为原始消息字符串的键值对，如：`{"key": "key", "value": "value", "message": "{\"key\": \"key\", \"value\": \"value\"}"}`
      - 若改字符串不为json格式，则按照plain类型进行处理
  - 必选：否
  -字段类型：String
  - 默认值：plain

<br/>

<a name="1LBc2"></a>
## 四、配置示例
```json
{
  "job": {
    "content": [{
      "reader": {
        "parameter" : {
          "broker" : "tcp://localhost:1883",
          "topic" : "test",
          "username" : "admin",
          "password" : "public",
          "isCleanSession": true,
          "qos": 2,
          "codec": "plain",
          "column" : [ {
            "name": "message",
            "type" : "string"
          }]
        },
        "name" : "emqxreader"
      },
      "writer" : {
        "parameter" : {
          "print" : true
        },
        "name" : "streamwriter"
      }
    } ],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 100
      },
      "restore": {
        "maxRowNumForCheckpoint": 0,
        "isRestore": false,
        "isStream" : true,
        "restoreColumnName": "",
        "restoreColumnIndex": 0
      },
      "log" : {
        "isLogger": false,
        "level" : "debug",
        "path" : "",
        "pattern":""
      }
    }
  }
}
```
<br />
