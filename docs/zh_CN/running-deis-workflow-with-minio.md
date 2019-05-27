# 使用MinIO运行Deis Workflow [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

[Deis Workflow](https://deis.com/)是一个开源的[PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) ，可以很容易地使我们在自己的服务器上部署和管理应用程序。Workflow建立于[Kubernetes](http://kubernetes.io/)和[Docker](https://www.docker.com/)的基础之上，提供一个具有[Heroku](https://www.heroku.com/)激发工作流的轻量级的PaaS。Workflow有多个模块化比较好的组件（请看 <https://github.com/deis>），它们之间使用Kubernetes system和一个对象存储服务进行通信。它可以配置为使用多种云存储服务，像[Amazon S3](https://aws.amazon.com/s3/), [Google Cloud Storage](https://cloud.google.com/storage/) ， [Microsoft Azure Storage](https://azure.microsoft.com/en-us/services/storage/)，当然，还有MinIO。我们目前不会建议你在Deis Workflow生产环境上使用MinIO，目前MinIO可以做为一个快速安装用于演示、开发、测试等的Deis Workflow集群的极好方法。事实上，我们默认提供了装有MinIO的Deis Workflow。

要使用它，请按照<https://docs-v2.readthedocs.io/en/latest/installing-workflow/>中的说明进行操作。完成安装后，请按以下三种方法进行部署：

- [Buildpack部署](https://docs-v2.readthedocs.io/en/latest/applications/using-buildpacks/)
- [Dockerfile部署](https://docs-v2.readthedocs.io/en/latest/applications/using-dockerfiles/)
- [Docker Image部署](https://docs-v2.readthedocs.io/en/latest/applications/using-docker-images/)

所有的三种部署方法，以及Workflow内部都广泛使用了MinIO：

- Buildpack部署使用MinIO存储代码和[slugs](https://devcenter.heroku.com/articles/slug-compiler)
- Dockerfile部署使用MinIO存储Dockerfiles和关联的artifacts
- Docker Image部署使用MinIO作为由Workflow运行的内部Docker registry的后备存储
- Workflow的内部数据库存储用户登录信息，SSH密钥等。它将所有数据备份到MinIO。
