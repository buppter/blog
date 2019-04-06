---
title: macOS下给VScode安装Golang插件
date: 2019-04-06 13:33:55
categories: 编程
tags:
    - Golang
---
最近学习Golang，IDE选择了VScode，主要原因是VScode轻巧还有丰富的各种插件。但是在配置好环境之后，vscode提示我安装插件，却一直安装失败，在折腾了好久之后终于算是安装成功了，所以记录一下。
<!--more-->
这里不讲具体的Golang和VScode的具体安装过程，需要的可以百度其他博客的讲解。这里直接讲解如何安装其他插件。  

切换到`GOPATH`路径下  

```bash
cd $GOPATH/src
```

然后在src文件夹下新建两个文件夹以及子文件夹    
```bash
sudo mkdir -p github.com/golang
sudo mkdir -p golang.org/x
```

克隆GitHub上的tools工具包:  
```bash
cd $GOPATH/src/github.com/golang
git clone https://github.com/golang/tools.git tools
```
再将`tools`文件夹拷贝到`golang.org/x`文件夹下:  
```bash
# 首先在golang.org/x下新建一个tools的文件夹
mkdir %GOPATH/src/golang.org/x/tools

# 然后把刚刚下载的tools工具包下的文件都拷贝过去
cp -r $GOPATH/src/github.com/golang/tools/ $GOPATH/src/golang.org/x/tools/
```
然后返回到`$GOPATH`目录下，执行`go install`命令即可。  
```bash
cd $GOPATH
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install github.com/josharian/impl
go install github.com/rogpeppe/godef
go install github.com/sqs/goreturns
go install github.com/golang/lint/golint
go install github.com/cweill/gotests/gotests
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install github.com/josharian/impl
go install github.com/rogpeppe/godef
go install github.com/sqs/goreturns
go install github.com/cweill/gotests/gotests
```
可能还会提示`golint`安装失败，是因为`golint`在`tools`里不包括，单独下载下来安装就可以了  
```bash
cd $GOPATH/src/golang.org/x
git clone https://github.com/golang/lint.git

# 返回#GOPATH目录
go install golang.org\x\lint\golint
```
至此，所有的插件都安装完成。


