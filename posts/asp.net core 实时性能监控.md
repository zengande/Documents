# asp.net core 实时性能监控
使用InfluxDB、Grafana

## Dockerfile 运行 InfluxDB、Grafana
``` yml
influxdb:
image: influxdb
  ports:
    - "8086:8086"
    - "8083:8083"
  environment:
    - INFLUXDB_DB=TogetherAppMetricsDB
    - INFLUXDB_ADMIN_ENABLED=true
    - INFLUXDB_ADMIN_USER=admin
    - INFLUXDB_ADMIN_PASSWORD=admin
grafana:
  image: grafana/grafana
  ports:
    - "3000:3000"
```


## 配置 Grafana
1. 浏览器打开 `<本地ip>:3000`，使用默认账号登录
2. 添加数据源
在Configuration中点击Add data source按钮，输入你安装的InfluxDB数据库信息
3. 点击仪表板设置模板

## 在API网关中App.Metrics
1. 安装必要的nuget包
```
> App.Metrics
> App.Metrics.AspNetCore.Endpoints
> App.Metrics.AspNetCore.Reporting
> App.Metrics.AspNetCore.Tracking
> App.Metrics.Reporting.InfluxDB
```
2. ConfigureServices 配置

StartUp.cs
``` csharp
public void ConfigureServices(IServiceCollection services)
{
      ...

      services.AddOptions();
      services.Configure<AppMetricsOptions>(Configuration.GetSection("AppMetrics"));

      services.AddAppMetrics(Configuration);
      services.AddOcelot(Configuration);
}

 public void Configure(ILoggerFactory loggerFactory, IApplicationBuilder app, IHostingEnvironment env)
 {

      app.UseMetricsAllEndpoints();
      app.UseMetricsAllMiddleware();

      app.UseOcelot();
 }

```

ServiceCollectionExtensions.cs
``` csharp 
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddAppMetrics(this IServiceCollection services, IConfiguration configuration)
    {
        var options = services.BuildServiceProvider()
            .GetRequiredService<IOptions<AppMetricsOptions>>()?.Value;
        if (options?.IsOpen == true)
        {
            var uri = new Uri(options.ConnectionString);
            var metrics = AppMetrics.CreateDefaultBuilder().Configuration.Configure(opt =>
            {
                opt.AddAppTag(options.App);
                opt.AddEnvTag(options.Env);
            }).Report.ToInfluxDb(opt =>
            {
                opt.InfluxDb.BaseUri = uri;
                opt.InfluxDb.Database = options.DatabaseName;
                opt.InfluxDb.UserName = options.UserName;
                opt.InfluxDb.Password = options.Password;
                opt.HttpPolicy.BackoffPeriod = TimeSpan.FromSeconds(30);
                opt.HttpPolicy.FailuresBeforeBackoff = 5;
                opt.HttpPolicy.Timeout = TimeSpan.FromSeconds(10);
                opt.FlushInterval = TimeSpan.FromSeconds(5);
            }).Build();

            services.AddMetrics(metrics);
            services.AddMetricsReportScheduler();
            services.AddMetricsTrackingMiddleware();
            services.AddMetricsEndpoints();
        }
        return services;
    }
}

public class AppMetricsOptions
{
    public bool IsOpen { get; set; }
    public string DatabaseName { get; set; }
    public string ConnectionString { get; set; }
    public string UserName { get; set; }
    public string Password { get; set; }
    public string App { get; set; }
    public string Env { get; set; }
}
```

## 最终效果
![avatar](/files/appmetrics-1.png)

## 参考文章
1. [.NET Core微服务之基于App.Metrics+InfluxDB+Grafana实现统一性能监控](https://www.cnblogs.com/edisonchou/p/integrated_performance_monitoring_foundation.html)
1. [ASP.NET Core Real-time Performance Monitoring](https://al-hardy.blog/2017/04/28/asp-net-core-monitoring-with-influxdb-grafana/)
2. [influxdb docker 文档](https://docs.docker.com/samples/library/influxdb/)
3. [grafana 官方文档](http://docs.grafana.org/installation/docker/)