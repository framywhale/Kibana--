# Kibana在Ubuntu下安装及简单的使用
简短地介绍了Kibana在Linux下的安装及使用。
`data.json`和`shakespeare.json`是本文档中将使用到的数据。

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

----------
本次安装的操作系统环境为**Ubuntu 14.04 64bit**

## Kibana的使用
### 如何打开Kinaba图形界面？
**PS:** Kibana运行在**5601**端口

  打开浏览器，在地址端输入：`http://YourDomain:5601`
  
  比如，一般使用的时候输入的本地端口：`http://localhost:5601`

### 如何将数据导入Elasticsearch？
在导入`.json`数据集之前，我们需要为各个字段建立一个**映射**。
【映射把索引里的文档划分成逻辑组，指明字段的特征，如字段是否可被搜索、是否被标记、是否能被拆分成多个文字等】

#### 1.构建映射：

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

需要构建的映射为：[在终端中输入如下命令]

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
**解释：**
* `-XPUT`之后的链接中`/shakespeare`意为在`localhost`上建立一个名为`shakespeare`的索引；
* `speaker`是一个类型为`string`的字段，`line_id`是一个类型为`整型`的字段，其余字段类似；
* `properties`中的每一个字段，都对应`shakespeare`原始数据的字段；

#### 2. 使用Elasticsearch的批量导入**API接口**来输入数据
**API接口**的使用格式如下：

``` shell
# 导入data数据
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @data.json  
# 导入shakespeare数据
curl -XPOST 'localhost:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json  
```

解释：
* 在`1.构建映射`后，我们才能将数据导入**建立好**的**索引**，如`shakespeare`。
* `_bulk?pretty`是Elasticsearch提供的API接口
* `@`之后接的是将要导入的数据文件
* 注意：
  * 必须要先正确建立索引，才能成功地将数据导入该索引；

#### 3. 验证数据是否成功导入
在终端中输入如下命令：
`curl 'localhost:9200/_cat/indices?v' `

你将看到类似下面的信息：
``` javascript
health status index               pri rep docs.count docs.deleted store.size pri.store.size
yellow open   data                  5   1       1000            0    420.1kb        420.2kb
yellow open   shakespeare           5   1     120111            0     16.0mb         18.9mb
```

以上例子中使用的数据，在[这里](https://github.com/framywhale/Kibana--)。

关于如何在Kibana中**构建索引**以及**Visualize**和**Dashboard**的操作，请见[视频](https://pan.baidu.com/s/1pLOBxkn)。

关于Kibana的更多使用详情，请参见[Kibana的中文文档](https://kibana.logstash.es/content/kibana/v5/setup.html)。

以上文档参考了博客：[《Kibana的基本使用》](http://blog.csdn.net/ming_311/article/details/50619859)。
