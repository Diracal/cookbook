# 如何用Prometheus监控MinIO[![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

[Prometheus](https://prometheus.io)是一个云原生监控平台，最初在SoundCloud上构建。Pormetheus提供一个多维的数据模型，其中包含了用指标名称和键值对标识的时间序列。数据的收集通过HTTP上的pull模型进行。pull数据的目标可由服务发现或者静态配置得到。

MinIO将Prometheus兼容数据作为未授权端点输出到`/minio/prometheus/metrics`中。希望监控其MinIO实例的用户可以指示Prometheus配置从该端点爬取数据。

本文解释如何设置Prometheus并且配置它从MinIO server中爬取数据。

## 前提条件

MinIO server发布`RELEASE.2018-05-11T00-29-24Z`或更高版本。为了启动MinIO，参考[MinIO快速启动](https://docs.min.io/docs/minio-quickstart-guide))。按照以下的步骤用Prometheus启动MinIO监控。

### 1. 下载Prometheus

平台上[下载最新的版本](https://prometheus.io/download)的Prometheus, 然后将它解压

```sh
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

Prometheus server是一个叫做`prometheus`的单个二进制文件（或Microsoft Windows上的`prometheus`）。运行二进制文件并且传递`--help`参数查看可选选项。


```sh
./prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server

. . .

```

参考[Prometheus文献](https://prometheus.io/docs/introduction/first_steps/)获得更多信息。

### 2. Prometheus配置

Prometheus配置用YAML编写的。添加MinIO server详细信息到位于`scrape_configs`部分下的配置文件中。

```yaml
scrape_configs:
  - job_name: minio
    metrics_path: /minio/prometheus/metrics
    static_configs:
      - targets: ['localhost:9000']
```

注意`localhost:9000`是MinIO server的示例地址，你需要将它更改为你配置文件中的适当的值。

### 3. 启动Prometheus

启动Prometheus，可通过运行

```sh
./prometheus --config.file=prometheus.yml
```

这里`prometheus.yml`是配置文件的名称。你现在可以查看Promehteus仪表板中的MinIO指标。在默认情况下Prometheus仪表板可在`http://localhost:9000`被访问。

## 公开的MinIO指标列表

MinIO server在`/minio/prometheus/metrics`端点公开了下面的指标。所有的这些指标都可以在Prometheus仪表板上访问。公开的指标及其定义的全部列表都可以在demo server的https://play.min.io:9000/minio/prometheus/metrics上得到。

- 以`go_`为前缀的标准运行指标
- 以`process_`为指标的运行水平指标
- 以`promhttp`为前缀的Prometheus爬取指标

- `minio_disk_storage_used_bytes` : 被当前MinIO server示例使用掉的磁盘存储的总byte数。
- `minio_http_requests_duration_seconds_bucket` : 不同时间段中所有请求类型（HEAD / GET / PUT / POST / DELETE）的累积计数器
- `minio_http_requests_duration_seconds_count` : 当前观察（即HTTP请求（HEAD/GET/PUT/POST/DELETE））次数的计数
- `minio_http_requests_duration_seconds_sum` : 服务所有HTTP请求（HEAD/GET/PUT/POST/DELETE）的当前合计时间(以秒为单位)
- `minio_network_received_bytes_total` : 由当前MinIO server实例接收到的总byte数量
- `minio_network_sent_bytes_total` : 由当前MinIO server实例发送的总byte数量
- `minio_offline_disks` : 当前MinIO server实例的脱机磁盘总数
- `minio_total_disks` : 当前MinIO server实例的总磁盘总数
- `minio_disk_storage_available_bytes` : MinIO server当前可用的存储空间总数（以byte为单位）
- `minio_disk_storage_total_bytes` : MinIO可用的总存储空间（以byte为单位）
- `process_start_time_seconds` : 自unix创立以来的MinIO server的启动时间（以秒为单位）

如果正在运行MinIO网关，磁盘/存储信息是非公开的。只有下列的指标是可用的。

- `minio_http_requests_duration_seconds_bucket` : 不同时间段中所有请求类型（HEAD / GET / PUT / POST / DELETE）的累积计数器
- `minio_http_requests_duration_seconds_count` : 当前观察（即HTTP请求（HEAD/GET/PUT/POST/DELETE））次数的计数
- `minio_http_requests_duration_seconds_sum` : 服务所有HTTP请求（HEAD/GET/PUT/POST/DELETE）的当前合计时间(以秒为单位)
- `minio_network_received_bytes_total` : 由当前MinIO server实例接收到的总byte数量
- `minio_network_sent_bytes_total` : 由当前MinIO server实例接收到的总byte数量
- `process_start_time_seconds` : 自unix创立以来的MinIO server的启动时间（以秒为单位）


对于启用了[`caching`](https://github.com/minio/minio/tree/master/docs/disk-caching)的MinIO实例，这些额外的指标是可用的。

- `minio_disk_cache_storage_bytes` : 当前MinIO server实例可用的缓存容量的总byte数
- `minio_disk_cache_storage_free_bytes` : 当前MinIO server实例可用的空闲缓存的总byte数
