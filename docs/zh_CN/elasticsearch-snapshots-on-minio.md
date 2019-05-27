# MinIO Server上的Elasticsearch快照 [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

Elasticsearch是一个分布式的RESTful和分析引擎。在本篇文章中，我们将要看到如何将Elasticsearch `6.x`快照存储到MinIO服务器中。


## 1. 前提条件

- 从[这里](https://docs.min.io/docs/minio-quickstart-guide)安装MinIO服务器。
- 从[这里](https://www.elastic.co/downloads/elasticsearch)下载`6.2.4`版本的Elasticserach。

## 2. 安装

在你想要安装Elasticsearch的文件夹里解压`elasticsearch-6.2.4`压缩包。


## 3. 在Elasticsearch安装S3 Repository插件

S3 repository插件添加了对对使用S3作为快照/恢复存储库的支持。用插件管理器安装插件。从Elasticsearch目录中运行以下命令

```sh
sudo bin/elasticsearch-plugin install repository-s3
```

## 4. 配置Elasticsearch

S3 Repository插件在默认情况下指向AWS S3。添加下面的字段到`conf/elasticsearch.yml`文件中，以便S3 Repository插件可以与MinIO安装进行通信。


```yaml
s3.client.default.endpoint: "http://127.0.0.1:9000" # Replace with actual MinIO server endpoint
s3.client.default.protocol: http                    # Replace with actual protocol (http/https)
```

S3 repository凭据非常敏感，必须被存储在elasticsearch秘钥库中。使用下面的命令为访问秘钥和秘钥创建条目。

```sh
bin/elasticsearch-keystore add s3.client.default.access_key
bin/elasticsearch-keystore add s3.client.default.secret_key
```

## 5. 配置MinIO server

假设MinIO server正在`http://127.0.0.1:9000`上运行。在你的MinIO server上创建一个叫做`elasticsearch`的存储桶。这个存储桶用来存储Elasticsearch快照。使用`mc`

```sh
mc config host add myminio http://127.0.0.1:9000 minio minio123
mc mb myminio/elasticsearch
```


## 6. 启动Elasticsearch并创建repository

启动Elasticsearch，可通过运行

```sh
bin/elasticsearch
```

Then create the repository by

```sh
curl -X PUT "localhost:9200/_snapshot/my_minio_repository" -H 'Content-Type: application/json' -d'
{
  "type": "s3",
  "settings": {
    "bucket": "elasticsearch",
    "endpoint": "http://127.0.0.1:9000",
    "protocol": "http"
  }
}
'
```

确认repository是否成功创建，可通过运行

```sh
curl -X GET http://127.0.0.1:9200/_snapshot/my_minio_repository?pretty
```

## 7. 创建快照（snapshots）

我们现在已经准备好创建快照。创建快照可通过使用

```sh
curl -X PUT http://127.0.0.1:9200/_snapshot/my_minio_repository/snapshot_1/\?wait_for_completion=true
```

这会创建一个快照并且将它上传到我们刚创建的MinIO repository中。可通过以下命令校验：

```sh
curl -X GET http://127.0.0.1:9200/_snapshot/my_minio_repository/snapshot_1/
```
