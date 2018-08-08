# asp.net core 使用 docker 中的 Exceptionless 实现日志记录

## 在 docker 中运行 Exceptionless
`docker-compose.yml`的内容：
``` yml
version: '3.4'

services:
  api:
    depends_on:
      - elasticsearch
    image: exceptionless/api-ci:latest
    environment:
      AppMode: Development
      ConnectionStrings__Elasticsearch: http://elasticsearch:9200
      ConnectionStrings__Redis: redis,abortConnect=false
      RunJobsInProcess: 'false'
      StorageFolder: /app/storage
    ports:
      - 5000:80
    volumes:
      - appdata:/app/storage

  jobs:
    depends_on:
      - api
    image: exceptionless/job-ci:latest
    command: run-all
    environment:
      AppMode: Development
      ConnectionStrings__Elasticsearch: http://elasticsearch:9200
      ConnectionStrings__Redis: redis,abortConnect=false
      StorageFolder: /app/storage
    volumes:
      - appdata:/app/storage

  ui:
    image: exceptionless/ui-ci:latest
    environment:
      AppMode: Development
      BaseUrl: http://ex-api.localtest.me:5000
    ports:
      - 5100:80

  elasticsearch:
    image: slideroom/elasticsearch:98
    environment:
      bootstrap.memory_lock: 'true'
      discovery.type: single-node
      ES_JAVA_OPTS: '-Xms512m -Xmx512m'
      xpack.security.enabled: 'false'
    ports:
      - 9200:9200
      - 9300:9300
    ulimits:
      memlock:
        soft: -1 
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data

  # kibana:
  #   depends_on:
  #     - elasticsearch
  #   image: slideroom/kibana:116
  #   ports:
  #     - 5601:5601

  redis:
    image: redis:alpine
    ports:
      - 6379:6379
 
volumes:
  esdata:
    driver: local
  appdata:
    driver: local
```
在 `docker-compose.yml` 文件目录下执行 `docker-compose up` 命令。
运行成功后可以在 <本地ip>:5100 访问UI

## 使用本地 Exceptionless 的配置
1. 创建新的账号并登录 Exceptionless。
2. 创建项目，获得 api 密钥。
3. 引用 nuget包，程序包管理器控制台执行 `Install-Package Exceptionless.AspNetCore`，也可以从 Nuget 上安装强名称程序包 `Exceptionless.AspNetCore.Signed`。
4. appSettings.json 的配置
``` json
"Exceptionless": {
    // api 密钥
    "ApiKey": "xDCAoTAteqTbscdJeBDCjMoKbshQYUyC7khhKFhc",
    // api 暴露的地址
    "ServerUrl": "http://localhost:5000",
    "DefaultData": {
      "JSON_OBJECT": "{ \"Name\": \"Blake\" }",
      "Boolean": true,
      "Number": 1,
      "Array": "1,2,3"
    },
    "DefaultTags": [ "xplat" ],
    "Settings": {
      "FeatureXYZEnabled": false
    }
 }
 ```
5. StartUp.cs 的配置。
``` csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory) 
{
    app.UseExceptionless(Configuration);
    //OR
    //app.UseExceptionless(new ExceptionlessClient(c => c.ReadFromConfiguration(Configuration)));
    //OR
    // app.UseExceptionless("W79xLqh2B79nm7LZvfAlH47Fxggk9tAaoIvJH3A0");
    //OR
    //loggerFactory.AddExceptionless("API_KEY_HERE");
    //OR
    //loggerFactory.AddExceptionless((c) => c.ReadFromConfiguration(Configuration));

    loggerFactory.AddExceptionless();
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();
    app.UseMvc();
}
 ```


以上配置就可以自动将所有未处理异常发送到 Exceptionless 了，也可以通过 `ex.ToExceptionless().Submit()` 向 Exceptionless 发送已处理的异常。


## 参考
1. [.NET Core微服务之基于Exceptionless实现分布式日志记录 - Edison Chou](http://www.cnblogs.com/edisonchou/p/exceptionless_foundation_and_quick_start.html)
2. [Exceptionless 官方示例](https://github.com/exceptionless/Exceptionless.Net/tree/master/samples)
