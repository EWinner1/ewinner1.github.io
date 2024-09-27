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

在实际的开发过程中，会将一切的**层**都抽象为**模块**，也就是说不存在 Service **层**等，只存在 Services **模块**整体作为模块存在，表现为极高的高内聚，低耦合，层与层的关系变更为模块和模块之间的依赖关系，学习 ABP 就是学习对于模块的使用。

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

详细的方法如下图
![ABP](/images/ABP-module-load.png)
通常情况下，我们可以在`ConfigureServices`和`OnApplicationInitialization`两个方法中进行对应用的一些常见配置。

## 依赖注入

在 ABP 框架中，依赖加载的顺序是按照模块的依赖顺序的反向进行加载，即执行的是先进后出的栈的加载顺序。

## 启动流程

通过`builder.Services.AddApplication<ModuleName>()`指定启动对象，并将与 ABP 相关的框架模块等自动注入到主程序中。

- 注册 ASP.Net Core 的基础服务，ABP 的核心服务
- 对所有的 ABP 模块进行依赖排序后加载
- 遍历所有模块，执行每一个模块的**配置服务**和**初始化**方法

## ABP 和 DDD 领域驱动设计

领域驱动设计（Domain-Driven Design，简称 DDD），是一种面向对象设计思想，它将软件系统分解为多个领域，每个领域都对应一个实体，实体之间通过行为进行交互，行为是领域内的规则，这些规则是领域内的核心，是领域驱动设计的核心。ABP 框架是 DDD 的最佳实践之一。ABP 框架的领域驱动设计，将领域模型抽象为模块，每个模块都对应一个实体，实体之间通过行为进行交互，行为是领域内的规则，这些规则是领域内的核心，是领域驱动设计的核心。
DDD 的四层架构表现为
├── 表示层
│ ├── .Web
│ ├── .HttpApi
│ └── .HttpApiClient
├── 应用层
│ ├── .Application.Contracts
│ └── .Application
├── 领域层
│ ├── .Domain.Shared
│ └── .Domain
└── 基础层
└── .EntityFrameworkCore
Domain.Shared 通常用于定义存放公共的常量，枚举等领域辅助对象。不依赖于其他任何项目。Domain 是解决方案中的核心项目，其中主要包括实体，聚合根，领域服务，仓储接口等。
应用层中 Application.Contracts 主要包含应用服务接口和数据传输对象，为了分离业务层的接口和实现。Application 包括应用服务，实现应用服务接口。
表示层中主要包括控制器，大部分情况下不需要手动创建控制器，通过自动生成器生成即可。客户端代理也不需要手动创建，通过自动生成器生成即可。
![ABP依赖关系图](/images/ABP-Dependencies.png)

# 实战

## EF Core 配置

1. 配置数据库连接字符串，在 DBContext 中定义实体和数据库表的映射关系
2. 使用 EF Core 构建迁移代码。
3. 使用迁移代码生成数据库，准备初始数据。
4. 应用迁移和初始数据。

在构建映射关系时，我们有两种方案，一种是使用注解方式，一种是使用 FluentAPI 方式。关于注解方式，可以参考这个例子

```C#
public class Category : AuditedAggregateRoot<Guid>
{
	[Required]
	[StringLength(128)]
	public string Name { get; set; }
}
```

使用注解方式时有个局限性，当使用 EF Core 特有的注解时会强制要求引用 EF Core 的相关 nuget 包。所以我们采用 FluentAPI 的方式对实体进行配置。

```C# FluentAPI
protected override void OnModelCreating(ModelBuilder builder)
{
	......
	builder.Entity<Category>(p =>
	{
		p.ToTable("Categories");
		p.Property(x => x.Name)
			.HasMaxLength(CategoryConsts.MaxNameLength)
			.IsRequired();
		p.HasIndex(x => x.Name);
	});
	builder.ApplyConfiguration(new ProductConfiguration());
	......
}
```

{% notel blue 附：ProductConfiguration配置 %}

```C#
internal class ProductConfiguration : IEntityTypeConfiguration<Product>
{
	public void Configure(EntityTypeBuilder<Product> builder)
	{
		builder.ToTable("Products");
		builder.Property(x => x.Name).HasMaxLength(ProductConsts.MaxNameLength).IsRequired();
		builder.HasOne(x => x.Category).WithMany().HasForeignKey(x => x.CategoryId).OnDelete(DeleteBehavior.Restrict).IsRequired();
		builder.HasIndex(x => x.Name).IsUnique();
	}
}
```

{% endnotel %}
因为在创建项目时会自动生成一个数据库迁移文件，而我们添加了新的实体，所以需要重新生成一个迁移文件。在终端中 cd 到 EF core 目录下，执行命令`dotnet ef migrations add xxxxxxxx`。通常，我们只需要再执行一次`dotnet ef database update`命令，即可完成数据库的更新。但是这种方式只能迁移实体结构，不能进行数据的初始化。所以我们使用 ABP 的数据播种功能。

## 数据播种

在 EF Core 中我们可以创建一个数据种子类，在迁移的时候，ABP 会找到所有的数据种子类，完成数据的播种。ABP 在继承了以上功能的同时也做了其他的更高级的功能，允许我们使用更加复杂的逻辑。数据种子类一般位于 Domain 文件夹中的 Data 文件夹下。

```C#
class ProductManagementDataSeedContributor(
	IRepository<Category, Guid> categoryRepository,
	IRepository<Product, Guid> productRepository) : IDataSeedContributor, ITransientDependency
{
	private readonly IRepository<Category, Guid> categoryRepository = categoryRepository;
	private readonly IRepository<Product, Guid> productRepository = productRepository;

	public Task SeedAsync(DataSeedContext context)
	{
		.......
	}
}
```

ABP 会自动发现实现了`IDataSeedContributor`接口的类，并调用`SeedAsync`方法。每当我们执行迁移操作时都会执行数据播种。此时运行 DbMigrator 程序即可完成数据的播种。同时，继承了具有审计功能的实体，在播种时会自动填充审计信息。

## CRUD

首先，我们需要 DTO(Data Transfer Object)。DTO 我们一般定义在 Application.Contracts 下，继承`AuditedEntityDto`类。创建 IProductAppService 接口，继承`IApplicationService`接口。

```C#
pubilc interface IProductAppService : IApplicationService
{
	Task<PagedResultDto<ProductDto>> GetListAsync(PagedAndSortedResultRequestDto input);
}
```

创建 ProductAppService 实现类，继承项目创建时生成的基类和 IProductAppService 接口。然后逐一实现 CRUD 的具体方法。

```C#
public class ProductAppService(
	IRepository<Product, Guid> productRepository,
	IRepository<Category, Guid> categoryRepository,
	IUnitOfWorkManager unitOfWorkManager) : ManageSystemAppService, IProductAppService
{
	private readonly IRepository<Product, Guid> productRepository = productRepository;
	private readonly IRepository<Category, Guid> categoryRepository = categoryRepository;
	private readonly IUnitOfWorkManager unitOfWorkManager = unitOfWorkManager;

	async Task<PagedResultDto<ProductDto>> IProductAppService.GetListAsync(PagedAndSortedResultRequestDto input)
	{
		var queryable = await productRepository.WithDetailsAsync(x => x.Category);
		queryable = queryable.Skip(input.SkipCount).Take(input.MaxResultCount).OrderBy(input.Sorting ?? nameof(Product.Name));

		var products = await AsyncExecuter.ToListAsync(queryable);
		var count = await productRepository.GetCountAsync();

		return new PagedResultDto<ProductDto>(
			count,
			ObjectMapper.Map<List<Product>, List<ProductDto>>(products));
	}

	public async Task CreateAsync()
	{
		var product = ObjectMapper.Map<>();
		await productRepository.InsertAsync(product);
	}

	public async Task<ListResultDto<>> GetCategoriesAsync()
	{
		var categories = await categoryRepository.GetListAsync();
		var categoryLookupDtos = ObjectMapper.Map<List<Category>, List<CategoryLookupDto>>(categories);

		return new ListResultDto<CategoryLookupDto>(categoryLookupDtos);
	}

	public async Task<ProductDto> GetAsync(Guid id)
	{
		var product = await productRepository.GetAsync(id);
		return ObjectMapper.Map<Product, ProductDto>(product);
	}

	public async Task UpdateAsync(Guid id, CreateUpdateProductDto input)
	{
		var product = await productRepository.GetAsync(id);
		ObjectMapper.Map(input, product);
	}

	public async Task DeleteAsync(Guid id)
	{
		await productRepository.DeleteAsync(id);
	}

	[UnitOfWork(isTransactional: true)]
	public async Task TodoSomethingAsync()
	{
		using (var uow = unitOfWorkManager.Begin(
			requiresNew: true,
			isTransactional: true,
			timeout: 2000))
		{
			await productRepository.InsertAsync(new Product() { });
			await productRepository.InsertAsync(new Product() { });
			await uow.CompleteAsync();
		}
	}
}
```

{% note info %}
Notice: 在撰写的过程中，我们还注意到了提示`CrudAppService`，作为一个 ABP 的基类，它已经帮我们实现了 CRUD 的功能，我们只需要继承它，并实现我们自己的业务逻辑即可。具体内容有待探究。
{% endnote %}
`ObjectMapper.Map<>`方法在调用的时候会默认使用 AutoMapper，所以我们需要去自己去定义一个 AutoMapper 的配置文件。AutoMapper 可以自动将 DTO 和实体进行转换。在这个例子中，AutoMapper 会将 Product 的 Category.Name 自动映射到 ProductDto 的 string 类型的 CategoryName 上。

```C#
public ManageSystemApplicationAutoMapperProfile()
{
	/* You can configure your AutoMapper mapping configuration here.
     * Alternatively, you can split your mapping configurations
     * into multiple profile classes for a better organization. */

	CreateMap<Product, ProductDto>();
}
```

完成后编写测试类，对对应功能进行测试。

```C#
public abstract class ProductAppServiceTests<TStartupModule> : ManageSystemApplicationTestBase<TStartupModule>
	where TStartupModule : IAbpModule
{
	private readonly IProductAppService productAppService;
	private readonly PagedAndSortedResultRequestDto dto;

	public ProductAppServiceTests()
	{
		productAppService = GetRequiredService<IProductAppService>();
		dto = new PagedAndSortedResultRequestDto();
	}

	[Fact]
	public async Task Should_Get_Products_List()
	{
		var result = await productAppService.GetListAsync(dto);

		result.TotalCount.ShouldBe(4);
		result.Items.ShouldContain(x => x.Name.Contains("Xiaomi 13 Ultra black"));
	}
}
```

因为测试类中不能使用依赖注入的方法，所以我们使用 GetRequiredService 解决所需要的依赖关系，获取我们所需要的服务。

{% note warning %}
Notice: 在实际测试过程中发现，无法在新建的`ProductAppServiceTests`中完成测试。经调查发现，ABP 自带的`SampleAppServiceTests`示例在继承了`ManageSystemApplicationTestBase<TStartupModule>`之后，还在`EfCoreSampleAppServiceTests`中再次继承了`SampleAppServiceTests`，最后实现了对内部的测试方法的直接测试调用。因此我们也单独实现了对应的测试方法。在官方的测试示例中，采用了`Substitute.For<>()`方法获取服务，但是会导致空引用错误。[Implementing unit tests in EF Core and MongoDB](https://abp.io/docs/8.0/Testing#implementing-unit-tests-in-ef-core-and-mongodb)
{% endnote %}

## 自动 API Controller

在 ABP 框架中，实现了自动 API Controller，根据命名约定和相关的配置自动将服务公开为 API 端点。对于前后端分离的项目，我们可以访问其`api/Abp/ServiceProxyScript`，得到自动生成的经过 api client 封装的 API 接口。也可以访问`api/swagger/index.html`，得到对应的 Swagger 文档。
