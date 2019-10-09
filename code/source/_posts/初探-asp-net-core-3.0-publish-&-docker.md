---
title: 初探 ASP.NET Core 3.0 Publish 与 docker容器化
author: White Worker
date: 2019-10-09 13:43:00
tags:
  - ASP.NET Core 
  - 'C#'
  - Docker
categories:
  - ASP.NET Core
---
[ASP.NET Core 3.0发布]
ASP.NET Core 3.0发布后，发布的机制改变了，可以直接发布可执行文件，Docker容器化体积也变小了，在这里初探一下这些改变和变化
<b>注：Windows 和Docker publish 可能大小有些差异，但是整体趋势一致，这里偷懒使用的是windows发布 XD</b>
<!-- more -->

# New 一个最简单的项目和发布 
```shell
$dotnet new mvc --auth None
$dotnet publish -r centos.7-x64 -c Release
```
发布完后大小为96.5 MB
这里为了快速测试，直接发布后拷贝
```Dockerfile
FROM mcr.microsoft.com/dotnet/core/runtime:3.0

WORKDIR /app
COPY bin/Release/netcoreapp3.0/centos.7-x64/publish .

EXPOSE 5000
CMD ["./dotnet3"]
```
<b> 走一个</b>
```shell
docker build -t dotnet3:1.0.0 .
```
```shell
$docker images
```
| REPOSITORY | TAG   | SIZE  |
| :---------------------: | :------------: | :----------------: |
| dotnet3    | 1.0.0 | 290MB |

<b>还是那么臃肿，看看有没什么新特性可以改进一下</b>


# 改进
##  打开程序集链接分析
.NET core 3.0 SDK 随附了一种工具，可以通过分析 IL 并剪裁未使用的程序集来减小应用的大小。
dotnet3.csproj
```csproj
<PropertyGroup>
  <PublishTrimmed>true</PublishTrimmed>
</PropertyGroup>
```
或者在命令行加上
##  这里顺便加上单个文件发布和自包含
```shell
$dotnet publish -r centos.7-x64 -c Release /p:PublishSingleFile=true /p:PublishTrimmed=true --self-contained true
```
发布完后大小为56.7 MB
##  镜像对比
这个将docker镜像替换成自包含的镜像，大小减少很多
>看看这两个镜像的分层就知道，mcr.microsoft.com/dotnet/core/runtime:3.0 的第一二层就是mcr.microsoft.com/dotnet/core/runtime-deps:3.0
![两者镜像比较图](/images/imagescompare-001.png)
```Dockerfile
FROM mcr.microsoft.com/dotnet/core/runtime-deps:3.0

WORKDIR /app
COPY bin/Release/netcoreapp3.0/centos.7-x64/publish .

EXPOSE 5000
CMD ["./dotnet3"]
```
##  结果
```shell
docker build -t dotnet3:1.0.1 .
docker run --rm --name dotnet3 -it -p 5000:5000 dotnet3:1.0.1
```

|REPOSITORY         |        TAG          |        SIZE            |
|  :----:           | :----:              | :----:                 |
|dotnet3            |     1.0.1  |     170MB|
|dotnet3            |     1.0.0  |     290MB|





> 容器化相对于Dotnet Core2.2版本已经有很大进步了，但是相对于其他语言例如go语言，还是远远不足（golang build from scratch 静态编译只需 39M左右）

# 参考
[dotnet-core-3-0](https://docs.microsoft.com/zh-cn/dotnet/core/whats-new/dotnet-core-3-0) 