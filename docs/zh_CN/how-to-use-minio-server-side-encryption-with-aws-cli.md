# 如何使用aws-cli调用MinIO服务端加密

 MinIO支持采用客户端提供的秘钥（SSE-C）进行S3服务端加密。下面的部分介绍了用AWS命令行接口（`aws-cli`）来进行用户的服务端加密：
(`aws-cli`):
* [前提条件](#prerequisites)
* [使用aws-cli操作SSE-C](#use-sse-c-with-aws-cli)
* [与安全相关的注意事项](#security-notice)

## <a name="prerequisites"></a>1. 前提条件

一个客户端必须为SSE-C指定三个HTTP请求头：
* `X-Amz-Server-Side-Encryption-Customer-Algorithm`: 算法标识符。它必须设置为"AES256"。
* `X-Amz-Server-Side-Encryption-Customer-Key`: 加密密钥。它必须是一个256位Base64编码字符串。
* `X-Amz-Server-Side-Encryption-Customer-Key-MD5`: 加密密钥的MDS校验和。它必须设置为加密密钥的MD5总和。注意：MD5校验和是原始二进制密钥的MD5总和，而不是base64编码的密钥。


从[这里](https://docs.min.io/docs/how-to-secure-access-to-minio-server-with-tls)下载MinIO Server,并将它安装成带有**TLS**的服务。

注意一下，如果你使用的是自己签名的TLS证书，那么当你往MinIO Server上传对象时，像aws-cli或者是mc的工具就会报错。如果你想获得一个CA结构签名的TLS证书，请参考`Let's Encrypt`。自己签名的证书应该仅做为内部开发和测试。

## <a name="use-sse-c-with-aws-cli"></a>2. 用aws-cli操作SSE-C

这个部分介绍如何通过aws-li用客户提供的加密密钥(SSE-C)进行服务端加密。

### 2.1 安装aws-li

你可以通过使用[这里](https://docs.min.io/docs/aws-cli-with-minio)描述的步骤安装AWS命令行界面。

### 2.2 创建名为`my-bucket`的存储桶

```sh
aws --no-verify-ssl --endpoint-url https://localhost:9000 s3api create-bucket --bucket my-bucket
```

### 2.3 用SSE-C上传对象

下面的实例显示了如何上传一个名为`my-secret-diary`的对象，内容来自文件`~/my-diary.txt`。注意你应该使用你自己的加密密钥。

```sh
aws s3api put-object \
  --no-verify-ssl \
  --endpoint-url https://localhost:9000 \
  --bucket my-bucket --key my-secret-diary \
  --sse-customer-algorithm AES256 \
  --sse-customer-key MzJieXRlc2xvbmdzZWNyZXRrZXltdXN0cHJvdmlkZWQ= \
  --sse-customer-key-md5 7PpPLAK26ONlVUGOWlusfg== \
  --body ~/my-diary.txt
```

在这个实例中，一个带有自签名证书的本地MinIO server运行在https://localhost:9000。TLS证书校验可以用`--no-verify-ssl`跳过。如果一个MinIO server使用一个CA签名的证书，则不应包含`--no-verify-ssl`，否则aws-cli将不能接受任何证书。

### 2.4 显示对象信息
  你**必须**指定正确的SSE-C秘钥才能显示加密对象的元数据：

  ```sh
  aws s3api head-object \
  --no-verify-ssl \
  --endpoint-url https://localhost:9000 \
  --bucket my-bucket \
  --key my-secret-diary \
  --sse-customer-algorithm AES256 \
  --sse-customer-key MzJieXRlc2xvbmdzZWNyZXRrZXltdXN0cHJvdmlkZWQ= \
  --sse-customer-key-md5 7PpPLAK26ONlVUGOWlusfg==
  ```

### 2.5 下载一个对象

下面的例子展示了如何删除一个本地的副本文件然后通过从服务器下载来还原：

删除文件`my-diary.txt`的本地副本：  
  
```sh 
`rm ~/my-diary.txt` 
```

从服务器下载该文件来恢复概该文件：
   
```sh
aws s3api get-object \
--no-verify-ssl \
--endpoint-url https://localhost:9000 \
--bucket my-bucket \
--key my-secret-diary \
--sse-customer-algorithm AES256 \
--sse-customer-key MzJieXRlc2xvbmdzZWNyZXRrZXltdXN0cHJvdmlkZWQ= \
--sse-customer-key-md5 7PpPLAK26ONlVUGOWlusfg== \
~/my-diary.txt
```

## <a name="security-notice"></a>3.安全须知
 - 根据S3规范，MinIO server将拒绝任何通过不安全（非TLS）连接进行的SSE-C请求。这意味着SSE-C**必须是**TLS / HTTPS，而且SSE-C请求包含着加密密钥。
 - 如果通过非TLS连接进行SSE-C请求，则**必须**将SSE-C加密密钥视为受损。
 - 根据S3规范，SSE-C PUT操作返回的content-md5与上传对象的MD5-sum不匹配。
 - MinIO Server使用防篡改加密方案来加密对象，并且**不会保存**加密密钥。 这意味着您有责任保管好加密密钥。 如果你丢失了某个对象的加密密钥，你将会不能对该对象解除密钥。
 - MinIO Server期望SSE-C加密密钥是*高熵*的。加密密钥是**不是**密码。如果你想使用密码，请确保使用诸如Argon2，scrypt或PBKDF2的基于密码的密钥派生函数（PBKDF）来派生高熵密钥。

