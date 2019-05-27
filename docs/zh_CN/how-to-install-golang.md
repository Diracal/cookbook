# 如何安装Golang? [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

## Ubuntu (Xenial) 16.04

### 构建依赖

本文仅针对x86-64平台上的Ubuntu 16.04+。

##### 安装Git

```
$ sudo apt-get install git 
```

##### 安装Go 1.11+

从[https://golang.org/dl/](https://golang.org/dl/)下载Go 1.11+。

```
$ wget https://dl.google.com/go/go1.11.1.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go1.11.1.linux-amd64.tar.gz
```

##### 设置PATH

添加PATH到你的``~/.bashrc``目录中。

```
export PATH=$PATH:/usr/local/go/bin:${HOME}/go/bin
```

##### 执行Source命令

```
$ source ~/.bashrc
```

##### 测试

```
$ go env
$ go version
```

## OS X (El Capitan) 10.11

### Build Dependencies

本章节仅针对x86-64平台上的OS X El Capitan 10.11+。

##### 安装brew

从[brew.sh](http://brew.sh/)安装brew。

##### 安装Git

```
$ brew install git 
```

##### 安装Go 1.10+

使用`brew`安装golang二进制。

```
$ brew install go
```

##### 设置PATH


添加PATH到你的``~/.bashrc_profile``目录中。

```
export PATH=${HOME}/go/bin:$PATH
```

##### 执行Source命令

```
$ source ~/.bash_profile
```

##### 测试

```
$ go env
$ go version
```
