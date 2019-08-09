---
title: ASP.NET Core Serilog ElasticSearch Format
author: White Worker
date: 2019-08-09 13:35:53
tags:
  - ASP.NET Core
  - 'C#'
  - Log
  - ElasticSearch 
  - Serilog 
categories:
  - ASP.NET Core
---
[ASP.NET Core 使用Serilog Format ElasticSearch]
在使用微服务框架EFK收集ASP.NET Core日志的时候，默认Console.Log 打印出来的Log被ElasticSearch+Fluentd收集后颜色属性会产生乱码（低版本ES会直接报错)
本篇将介绍 使用Serilog的ElasticSearch Format收集日志
<!-- more -->
## 1. 安装NuGet包
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.ElasticSearch

## 2. Startup
Program.cs
```cs
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
    .UseSerilog((ctx, config) =>
{
    config
        .MinimumLevel.Information()
        .Enrich.FromLogContext();

    if (ctx.HostingEnvironment.IsDevelopment())
    {
        config.WriteTo.Console();
    }
    else
    {
        config.WriteTo.Console(new ElasticsearchJsonFormatter());
    }
})
```

## 参考
[writing-logs-to-elasticsearch-with-fluentd-using-serilog-in-asp-net-core](https://andrewlock.net/writing-logs-to-elasticsearch-with-fluentd-using-serilog-in-asp-net-core/) 