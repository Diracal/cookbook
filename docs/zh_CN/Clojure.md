# 如何使用Java的MinIO SDK操作Clojure

在本文中，我们将学习如何使用`minio-java`操作Clojure。
In this receipe we will learn how to use `minio-java` with Clojure.

## 前提条件
安装[`Clojure`](https://clojure.org/community/downloads)和[`Leiningen`](https://leiningen.org/)。

如果要检查，可以通过运行-
```
lein --help
```

## Clojure项目
下面是使用minio-java操作Clojure的步骤:


### 1. 设置
使用Leiningen创建一个新的Clojure项目。
```
lein new <templateName> <projectName>
```

示例：
```
lein new app clojproject
```

进入`project`查看`:dependencies`部分，它由项目所要求的依赖组成，以便装载、编译和运行。
```
lein deps
```
运行这个命令来安装成批的依赖到你的项目中：

现在使用Leiningen启动clojure - 
 
```
lein repl
```

测试这个项目需要运行`(-main)`。
```
user=> (-main)
Hello, World!
nil
```

或者你可以通过执行执行`lein run`命令来运行这个项目。

### 2. 从maven下载minio-java
转到[`maven`]（http://search.maven.org/）并搜索`minio`。
下载最新的版本并在`project.cli`中添加依赖。

```
::dependencies [
                [io.minio/minio "4.0.2"]
               ]
```
运行`lein deps`。

检查`lein classpath`来查看minio.jar是否存在。
```
/.m2/repository/io/minio/minio/4.0.2/minio-4.0.2.jar
```

重新启动Clojure的repl并使用minio—java。

### 3. 测试
使用你的工程项目中装载的类或方法。

```
(ns clojproject.core
 (import io.minio.MinioClient)
)

(defn -main
  [& args]
    ;; makeBucket
    (.makeBucket (new io.minio.MinioClient "https://play.min.io:9000" "Q3AM3UQ867SPQQA43P2F" "zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG") "testbucketclojure")
    
    ;;putObject
    (.putObject (new io.minio.MinioClient "https://play.min.io:9000" "Q3AM3UQ867SPQQA43P2F" "zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG") "testbucketclojure"  "objectname" "/Users/aarushiarya/Desktop/testFile")
    (println "Done.")
    )
    
```


