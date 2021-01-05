---
catalog: true  
title: 'ARM 机器编译 Go SDK'
sidebar:
  nav: go-sdk
---

# 环境说明

* 操作系统： CentOS Linux release 8.2.2004 (Core)
* 内核版本：Linux 10-23-173-45 4.18.0-193.el8.aarch64
* go 版本： [go1.15.6.linux-arm64.tar.gz](https://golang.google.cn/dl/go1.15.6.linux-arm64.tar.gz)

**注意：go源码选择 arm 平台的，而非 amd**

# 1. 环境准备

### 1.1 go 安装

* 下载解压
<div class="copyable" markdown="1">

  ```shell
  yum install -y  wget
  wget https://golang.google.cn/dl/go1.15.6.linux-arm64.tar.gz 
  tar -xzvf go1.15.6.linux-arm64.tar.gz  && mv go /usr/local/
  ```
</div>


* 添加环境变量

  打开文件 /etc/profile 末尾添加一下内容 
<div class="copyable" markdown="1">

  ```shell
   GOROOT
  export GOROOT=/usr/local/go
   GOPATH
  export GOPATH=/data/go
   GOPATH bin
  export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
  ```
</div>

  激活配置
<div class="copyable" markdown="1">
  ```shell
  source /etc/profile
  ```
</div>

### 1.2 git 安装配置
<div class="copyable" markdown="1">

  ```shell
  yum install git
  git config --global user.name "xxx"
  git config --global user.email "xxx@xxx"
  ```
</div>

## 2. 运行 SDK
<div class="copyable" markdown="1">

  ```shell
  go get github.com/ufilesdk-dev/ufile-gosdk
  cd example; go run demo_file.go
  ```
</div>

  ![image-20201209171357007](ARM%20%E6%9C%BA%E5%99%A8%E7%BC%96%E8%AF%91%20Go%20SDK.assets/image-20201209171357007.png)



