---
title: ABP framework
categories: [Tech]
date: 2024-09-14 09:47:07
tags: [Abp framework]
sticky:
thumbnail:
excerpt: ABP framework简述
---

# 简要

ABP framework，全名 ASP.NET Boierplate Project，是基于 ASP.Net Core，通过遵循软件开发最佳实践和最新技术创建现代 web 应用程序和 API。在后续发展过程中逐渐迭代并更名为 ABP framework。为了将企业级的开发技术与最先进的架构组合起来而诞生的快速开发套件。具有完善的基础设施，基于 DDD 领域驱动设计的分层架构，模块化开发。
相关链接：[Abp framework - Github](https://github.com/abpframework/abp) 和 [abp.io 官网](https://abp.io/)。
在开发过程中，对于熟悉 ABP 框架的人来说可以使用 ABP cli 进行快速创建，通过运行指令快速安装。

```Powershell
PS> dotnet tool install -g Volo.Abp.Studio.Cli
```

## 控制台程序开发

对于普通的控制台应用程序，我们只需要创建一个根模块，然后使其继承`AbpModule`即可完成根模块的创建。

```C# 根模块
internal class HelloABPModule : AbpModule
{
	......
}
```

之后我们可以配置一个简单的`Services`以供使用

```C# Services
internal class HelloWorldService
{
	public void run()
	{
		Console.WriteLine("Hello World");
	}
}
```

在控制台的主程序中写入

```C# 主程序
var app = AbpApplicationFactory.Create<HelloABPModule>();
app.Initialize();

var services = app.ServiceProvider.GetService<HelloWorldService>();
services.run();
```

修改根模块，引入`Services`即可完成运行

```C# 根模块
internal class HelloABPModule : AbpModule
{
	public override void ConfigureServices(ServiceConfigurationContext context)
	{
		context.Services.AddTransient<HelloWorldService>();
		base.ConfigureServices(context);
	}
}
```

对于`Services`模块，我们也可以通过让其继承`ITransientDependency`接口实现对于普通的服务的自动化依赖注入。

## MVC Web 程序开发

对于 MVC Web 的开发和控制台程序比较相似，只不过继承的包不同于控制台程序。
同样需要先创建一个根模块，然后在主程序中创建根模块，只在根模块的继承上有些许不同。

```C# 主程序
using HelloABP.Web.Modules;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddApplication<HelloABPWebModule>();

var app = builder.Build();
app.InitializeApplication();
app.Run();
```

```C# 根模块
[DependsOn(typeof(AbpAspNetCoreMvcModule))]
public class HelloABPWebModule : AbpModule
{
	......
}
```

## 生命周期

在模块中的几个重要的生命周期

```C#
[DependsOn(typeof(AbpAspNetCoreMvcModule))]
public class HelloABPWebModule : AbpModule
{
	//按顺序加载，配置
	public override void PreConfigureServices(ServiceConfigurationContext context)
	{
		base.PreConfigureServices(context);
	}

	public override void ConfigureServices(ServiceConfigurationContext context)
	{
		base.ConfigureServices(context);
	}

	public override void PostConfigureServices(ServiceConfigurationContext context)
	{
		base.PostConfigureServices(context);
	}

	public override void OnPreApplicationInitialization(ApplicationInitializationContext context)
	{
		base.OnPreApplicationInitialization(context);
	}

	public override void OnApplicationInitialization(ApplicationInitializationContext context)
	{
		base.OnApplicationInitialization(context);
	}

	public override void OnPostApplicationInitialization(ApplicationInitializationContext context)
	{
		base.OnPostApplicationInitialization(context);
	}

	public override void OnApplicationShutdown(ApplicationShutdownContext context)
	{
		base.OnApplicationShutdown(context);
	}
}
```
通常情况下，我们可以在`ConfigureServices`和`OnApplicationInitialization`两个方法中进行对应用的一些常见配置。