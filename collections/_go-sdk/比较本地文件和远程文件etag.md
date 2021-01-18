---
catalog: true  
title: '比较本地文件和远程文件etag'

---
SDK 提供`CompareFileEtag`方法比较本地文件和远程文件etag，适用于文件完整性判断，例如文件上传、下载过程中是否发生丢失。完整代码参见[Github](https://github.com/ufilesdk-dev/ufile-gosdk/blob/master/file.go)。

示例如下：

<div class="copyable" markdown="1">

```go
package main

import (
	ufsdk "github.com/ufilesdk-dev/ufile-gosdk"
	"log"
)

const (
	ConfigFile = "./config.json"
	FilePath = ""
	KeyName = ""
)

func main() {

	// 加载配置，创建请求
	config, err := ufsdk.LoadConfig(ConfigFile)
	if err != nil {
		panic(err.Error())
	}
	req, err := ufsdk.NewFileRequest(config, nil)
	if err != nil {
		panic(err.Error())
	}

	// 比较本地文件和远程文件etag,返回true则表明Bucket中存在KeyName文件且etag一致，返回false则表示Bucket不存在KeyName文件或KeyName文件存在但etag不一致
	ok := req.CompareFileEtag(KeyName, FilePath)
	if !ok {
		log.Printf("本地文件 %s 和远程文件 %s 不一致！",FilePath,  KeyName)
	} else {
		log.Printf("本地文件 %s 和远程文件 %s 一致！",FilePath,  KeyName)
	}
}

```
</div>
