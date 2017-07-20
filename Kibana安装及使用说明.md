# Kibana在Ubuntu下安装及简单的使用

## 安装和启动
参考至[elastic官网](https://www.elastic.co/guide/en/kibana/5.5/install.html)。
### 压缩包安装
1. **下载**对应平台的[二进制安装包](https://www.elastic.co/guide/en/kibana/5.5/targz.html)
2. **解压**`.zip`或者`.tar.gz`压缩文件
3. 在安装目录里**运行**: `bin/kibana (Linux/MacOSX)` 或 `bin\kibana.bat (Windows)`

### Ubuntu的apt-get安装
#### 1. **下载并安装**公共密钥：
   * `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`
#### 2. 从APT-repository**安装**：
   * `sudo apt-get install apt-transport-https`
   * `echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list`
   * `sudo apt-get update && sudo apt-get install kibana`

#### 3. 启动及关闭：
* 启动之前，需要首先在终端中输入命令 `ps -p 1` ，来确定你的系统使用的是 **SysV init** 还是 **systemd**:
 
* 查看终端上输出的`CMD`一栏，选择**对应**的启动和关闭方式
 
* `init` :    
  * 启动：`sudo -i service kibana start`
  * 关闭：`sudo -i service kibana stop`
* `systemd`:    
  * 启动：`sudo systemctl start kibana.service`
  * 关闭：`sudo systemctl stop kibana.service`


## Kibana的使用
**PS:** Kibana运行在**5601**端口
### 将数据导入Elasticsearch
在导入`.json`数据集之前，我们需要为各个字段建立一个**映射**。
【映射把索引里的文档划分成逻辑组，指明字段的特征，如字段是否可被搜索、是否被标记、是否能被拆分成多个文字等】

以`shakespeare.json`数据集为例，
`shakespeare`数据集的组织格式为：
``` javascript
  {  
    "line_id": INT,  
    "play_name": "String",  
    "speech_number": INT,  
    "line_number": "String",  
    "speaker": "String",  
    "text_entry": "String",  
  } 
```

#### 1.构建映射：
需要构建的映射为：
[在终端中输入如下命令]
``` shell
curl -XPUT http://localhost:9200/shakespeare -d '  
{  
 "mappings" : {  
  "_default_" : {  
   "properties" : {  
    "speaker" : {"type": "string", "index" : "not_analyzed" },  
    "play_name" : {"type": "string", "index" : "not_analyzed" },  
    "line_id" : { "type" : "integer" },  
    "speech_number" : { "type" : "integer" }  
   }  
  }  
 }  
}  
'; 
```
以上映射意思为， `speaker`是一个类型为`string`的字段，`line_id`是一个类型为`整型`的字段，其余字段类似。

#### 2. 使用Elasticsearch的批量导入**API**来输入数据
``` shell
# 导入data数据
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @data.json  
# 导入shakespeare数据
curl -XPOST 'localhost:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json  
```

#### 3. 验证数据是否成功导入
在终端中输入如下命令：
`curl 'localhost:9200/_cat/indices?v' `
你将看到类似下面的信息：
``` javascript
health status index               pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank                  5   1       1000            0    418.2kb        418.2kb
yellow open   shakespeare           5   1     111396            0     17.6mb         17.6mb
yellow open   logstash-2015.05.18   5   1       4631            0     15.6mb         15.6mb
yellow open   logstash-2015.05.19   5   1       4624            0     15.7mb         15.7mb
yellow open   logstash-2015.05.20   5   1       4750            0     16.4mb         16.4mb
```
关于如何在Kibana中构建索引以及Visualize操作的教程，见视频。

请参见Kibana的中文文档，请点击[这里](https://kibana.logstash.es/content/kibana/v5/setup.html)。

以上文档参考了博客：[《Kibana的基本使用》](http://blog.csdn.net/ming_311/article/details/50619859)。
