# Oracle Reader

<a name="c6v6n"></a>
## 一、插件名称
名称：**oraclereader**<br />
<a name="jVb3v"></a>
## 二、支持的数据源版本
**Oracle 9 及以上**<br />
<a name="2lzA4"></a>
## 三、参数说明<br />

- **connection**
  - 描述：数据库连接参数，包含jdbcUrl、schema、table等参数
  - 必选：是
  - 字段类型：List
    - 示例：指定jdbcUrl、schema、table
  ```json
  "connection": [{
       "jdbcUrl": ["jdbc:oracle:thin:@0.0.0.1:1521:oracle"],
       "table": ["table"],
       "schema":"public"
      }] 
   ```
  - 默认值：无

<br/>

- **jdbcUrl**
  - 描述：针对关系型数据库的jdbc连接字符串
<br />jdbcUrl参考文档：[Oracle官方文档](http://www.oracle.com/technetwork/database/enterprise-edition/documentation/index.html)
  - 必选：是
  - 字段类型：List
  - 默认值：无

<br/>

- **schema**
  - 描述：数据库schema名
  - 必选：否
  - 字段类型：String
  - 默认值：无

<br/>

- **table**
   - 描述：目的表的表名称。目前只支持配置单个表，后续会支持多表
   - 必选：是
   - 字段类型：List
   - 默认值：无
   
<br/> 

- **username**
  - 描述：数据源的用户名
  - 必选：是
  - 字段类型：String
  - 默认值：无

<br/>

- **password**
  - 描述：数据源指定用户名的密码
  - 必选：是
  - 字段类型：String
  - 默认值：无

<br/>

- **where**
  - 描述：筛选条件，reader插件根据指定的column、table、where条件拼接SQL，并根据这个SQL进行数据抽取。在实际业务场景中，往往会选择当天的数据进行同步，可以将where条件指定为gmt_create > time。
  - 注意：不可以将where条件指定为limit 10，limit不是SQL的合法where子句。
  - 必选：否
  - 字段类型：String
  - 默认值：无

<br/>

- **splitPk**
  - 描述：当speed配置中的channel大于1时指定此参数，Reader插件根据并发数和此参数指定的字段拼接sql，使每个并发读取不同的数据，提升读取速率。
  - 注意:
    - 推荐splitPk使用表主键，因为表主键通常情况下比较均匀，因此切分出来的分片也不容易出现数据热点。
    - 目前splitPk仅支持整形数据切分，`不支持浮点、字符串、日期等其他类型`。如果用户指定其他非支持类型，FlinkX将报错！
    - 如果channel大于1但是没有配置此参数，任务将置为失败。
  - 必选：否
  - 字段类型：String
  - 默认值：无

<br/>

- **fetchSize**
  - 描述：一次性从数据库中读取多少条数据，jdbc默认一次将所有结果都读取到内存中，在数据量很大时可能会造成OOM，设置这个参数可以控制每次读取fetchSize条数据。
  - 注意：此参数的值不可设置过大，否则会读取超时，导致任务失败。
  - 必选：否
  - 字段类型：int
  - 默认值：1000

<br/>

- **queryTimeOut**
  - 描述：查询超时时间，单位秒。
  - 注意：当数据量很大，或者从视图查询，或者自定义sql查询时，可通过此参数指定超时时间。
  - 必选：否
  - 字段类型：int
  - 默认值：3000

<br/>

- **customSql**
  - 描述：自定义的查询语句，如果只指定字段不能满足需求时，可通过此参数指定查询的sql，可以是任意复杂的查询语句。
  - 注意：
    - 只能是查询语句，否则会导致任务失败；
    - 查询语句返回的字段需要和column列表里的字段对应；
    - 当指定了此参数时，connection里指定的table无效；
    - 当指定次参数时，column必须指定具体字段信息，不能以*号代替；
  - 必选：否
  - 字段类型：String
  - 默认值：无

<br/>

- **column**
  - 描述：需要读取的字段。
  - 格式：支持3中格式
<br />1.读取全部字段，如果字段数量很多，可以使用下面的写法：
```bash
"column":["*"]
```

  - 2.只指定字段名称：
```
"column":["id","name"]
```

  - 3.指定具体信息：
```json
"column": [{
    "name": "col",
    "type": "datetime",
    "format": "yyyy-MM-dd hh:mm:ss",
    "value": "value"
}]
```

  - 属性说明:
    - name：字段名称
    - type：字段类型，可以和数据库里的字段类型不一样，程序会做一次类型转换
    - format：如果字段是时间字符串，可以指定时间的格式，将字段类型转为日期格式返回
    - value：如果数据库里不存在指定的字段，则会把value的值作为常量列返回，如果指定的字段存在，当指定字段的值为null时，会以此value值作为默认值返回
  - 必选：是
  - 字段类型：List
  - 默认值：无

<br/>

- **polling**
  - 描述：是否开启间隔轮询，开启后会根据`pollingInterval`轮询间隔时间周期性的从数据库拉取数据。开启间隔轮询还需配置参数`pollingInterval`，`increColumn`，可以选择配置参数`startLocation`。若不配置参数`startLocation`，任务启动时将会从数据库中查询增量字段最大值作为轮询的开始位置。
  - 必选：否
  - 字段类型：Boolean
  - 默认值：false

<br/>

- **pollingInterval**
  - 描述：轮询间隔时间，从数据库中拉取数据的间隔时间，默认为5000毫秒。
  - 必选：否
  - 字段类型：long
  - 默认值：5000

<br/>

- **increColumn**
  - 增量字段，可以是对应的增量字段名，也可以是纯数字，表示增量字段在column中的顺序位置（从0开始）
  - 必选：否
  - 字段类型：String或int
  - 默认值：无
  
<br/>  

- **startLocation**
  - 描述：增量查询起始位置
  - 必选：否
  - 字段类型：String
  - 默认值：无
  
<br/>  

- **useMaxFunc**
  - 描述：用于标记是否保存endLocation位置的一条或多条数据，true：不保存，false(默认)：保存， 某些情况下可能出现最后几条数据被重复记录的情况，可以将此参数配置为true
  - 必选：否
  - 字段类型：Boolean
  - 默认值：false

<br/>

- **requestAccumulatorInterval**
  - 描述：发送查询累加器请求的间隔时间。
  - 必选：否
  - 字段类型：int
  - 默认值：2

<br/>

<a name="1LBc2"></a>
## 四、配置示例
<a name="tCNPA"></a>
#### 1、基础配置
```json
{
  "job": {
    "content": [
      {
        "reader": {
          "parameter": {
            "username": "username",
            "password": "password",
            "connection": [{
              "jdbcUrl": ["jdbc:oracle:thin:@0.0.0.1:1521:oracle"],
              "table": ["TABLE"]
            }],
            "column": ["*"],
            "customSql": "",
            "where": "ID < 10000",
            "splitPk": "",
            "fetchSize": 1024,
            "queryTimeOut": 1000,
            "requestAccumulatorInterval": 2
          },
          "name": "oraclereader"
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": true
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 1
      },
      "restore": {
        "maxRowNumForCheckpoint": 0,
        "isRestore": false,
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
<a name="NTZg1"></a>
#### 2、多通道
```json
{
  "job": {
    "content": [
      {
        "reader": {
          "parameter": {
            "username": "username",
            "password": "password",
            "connection": [{
              "jdbcUrl": ["jdbc:oracle:thin:@0.0.0.1:1521:oracle"],
              "table": ["TABLE"]
            }],
            "column": ["*"],
            "customSql": "",
            "where": "ID < 10000",
            "splitPk": "id",
            "fetchSize": 1024,
            "queryTimeOut": 1000,
            "requestAccumulatorInterval": 2
          },
          "name": "oraclereader"
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": true
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 2,
        "bytes": 0
      },
      "errorLimit": {
        "record": 1
      },
      "restore": {
        "maxRowNumForCheckpoint": 0,
        "isRestore": false,
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
<a name="u6I8L"></a>
#### 3、指定`customSql`
```json
{
  "job": {
    "content": [
      {
        "reader": {
          "parameter": {
            "username": "username",
            "password": "password",
            "connection": [{
              "jdbcUrl": ["jdbc:oracle:thin:@0.0.0.1:1521:oracle"],
              "table": ["table"]
            }],
            "column": ["ID","USER_ID","NAME"],
            "customSql": "select * from table where ID > 20",
            "where": "ID < 10000",
            "splitPk": "ID",
            "fetchSize": 1024,
            "queryTimeOut": 1000,
            "requestAccumulatorInterval": 2
          },
          "name": "oraclereader"
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": true
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 1
      },
      "restore": {
        "maxRowNumForCheckpoint": 0,
        "isRestore": false,
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
<a name="RcDKX"></a>
#### 4、增量同步指定`startLocation`
```json
{
  "job": {
    "content": [
      {
        "reader": {
          "parameter": {
            "username": "username",
            "password": "password",
            "connection": [{
              "jdbcUrl": ["jdbc:oracle:thin:@0.0.0.1:1521:oracle"],
              "table": ["TABLE"]
            }],
            "column": [{
              "name": "ID",
              "type": "NUMBER"
            },{
              "name": "USER_ID",
              "type": "NUMBER"
            },{
              "name": "NAME",
              "type": "VARCHAR"
            }],
            "customSql": "",
            "where": "ID < 1600",
            "splitPk": "ID",
            "increColumn": "ID",
            "startLocation": "20",
            "fetchSize": 1024,
            "queryTimeOut": 1000,
            "requestAccumulatorInterval": 2
          },
          "name": "oraclereader"
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": true
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 1
      },
      "restore": {
        "maxRowNumForCheckpoint": 0,
        "isRestore": false,
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
<a name="gOu04"></a>
#### 5、间隔轮询
```json
{
  "job": {
    "content": [
      {
        "reader": {
          "parameter": {
            "username": "username",
            "password": "password",
            "connection": [{
              "jdbcUrl": ["jdbc:oracle:thin:@0.0.0.1:1521:oracle"],
              "table": ["TABLE"]
            }],
            "column": [{
              "name": "ID",
              "type": "NUMBER"
            },{
              "name": "USER_ID",
              "type": "NUMBER"
            },{
              "name": "NAME",
              "type": "VARCHAR"
            }],
            "customSql": "",
            "where": "ID > 1600",
            "splitPk": "ID",
            "increColumn": "ID",
            "startLocation": "20",
            "fetchSize": 1024,
            "queryTimeOut": 1000,
            "polling": true,
            "pollingInterval": 3000,
            "requestAccumulatorInterval": 2
          },
          "name": "oraclereader"
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": true
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 1
      },
      "restore": {
        "maxRowNumForCheckpoint": 0,
        "isRestore": false,
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
