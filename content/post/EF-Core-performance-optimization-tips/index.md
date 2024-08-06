---
title: "EF Core性能优化技巧"
description: 
slug: "EF-Core-performance-optimization-tips"
date: 2024-08-06
categories:
    - EFCore
---

## 使用实例池

EFCore2.0 为DbContext引入新的注册方式：透明地注册了 DbContext实例池，使用这种方式可以避免始终创建新的实例，EF Core 将重置其状态并将其存储在内部池中；当下次请求新的实例时，将返回该共用实例，而不是设置新的实例

使用示例：

```csharp
services.AddDbContext<HandshakesWebDBContext>(options => options.UseSqlServer(connectionConfiguration.WebDBConnection));
```

替换为：

```csharp
builder.Services.AddDbContextPool<HandshakesWebDBContext>(options => options.UseSqlServer(connectionConfiguration.WebDBConnection), poolSize: 80);
//注意设置最大连接数，一旦超过默认配置的连接池最大数量，会回退到按需创建实例的行为
```

基准测试(官方) [测试代码](https://github.com/dotnet/EntityFramework.Docs/blob/main/samples/core/Benchmarks/ContextPooling.cs)：

| 方法                  | 数量 | 平均值   | 错误     | 标准偏差 | Gen 0   | Gen 1 | Gen 2 | 已分配   |
| --------------------- | ---- | -------- | -------- | -------- | ------- | ----- | ----- | -------- |
| WithoutContextPooling | 1    | 701.6 us | 26.62 us | 78.48 us | 11.7188 | \-    | \-    | 50.38 KB |
| WithContextPooling    | 1    | 350.1 us | 6.80 us  | 14.64 us | 0.9766  | \-    | \-    | 4.63 KB  |

> 注意事项：虽然在大部分情况下这种做法对性能的提升可能并不是非常明显，但是这是一种好的实践方式，避免资源浪费的同时对性能带来一定的提升。

## 使用拆分查询

了解什么是 [笛尔卡乘积](https://www.wikiwand.com/en/Cartesian_product) ?

通俗地来讲指的是从两个集合(Set)中的元素组成新的配对集合 以麦当劳套餐来比喻,门店将汉堡线和饮品线上的每个产品集合组成一个新的套餐会有多少种套餐

在数据库中的表现形式正是联表查(join)操作 两个表在数据量不是很大的情况下查询来讲可能对性能影响模棱两可 但是对于一些因业务需求日益增加列的大宽表以及数据存量过大的表来讲就会产生查询过慢以及数据冗余的问题  
尤其适合一对多且子表数据量较大的场景。

看一段Linq代码：

```sql
var data = ctx.As
    .Include(x => x.Bs)
    .Include(x => x.Cs)
    .ThenInclude(x => x.D1s)
    .Include(x => x.Cs)
    .ThenIncude(x => x.C1s)
    .ThenInclude(x=>x.D2s)
    .ToList();
```

监控查看生成的Sql语句：

```sql
SELECT [A].[Id], [A].[Name], 
       [B].[Id], [B].[AId], [B].[Name],
       [C].[Id], [C].[AId], [C].[Name],
       [D1].[Id], [D1].[CId], [D1].[Name],
       [C1].[Id], [C1].[CId], [C1].[Name],
       [D2].[Id], [D2].[C1Id], [D2].[Name]
FROM [As] AS [A]
LEFT JOIN [Bs] AS [B] ON [A].[Id] = [B].[AId]
LEFT JOIN [Cs] AS [C] ON [A].[Id] = [C].[AId]
LEFT JOIN [D1s] AS [D1] ON [C].[Id] = [D1].[CId]
LEFT JOIN [C1s] AS [C1] ON [C].[Id] = [C1].[CId]
LEFT JOIN [D2s] AS [D2] ON [C1].[Id] = [D2].[C1Id]
```

毫无疑问，这一段糟糕的sql语句，假设每张表的数据量都很大的情况下，这对查询无疑是一种很大的负担，如果条件再复杂一点，对整个语句的分析也是很糟糕的。[关于阿里开发规范中定义超过3张表的join查询是被禁止的 (未查证)](https://cloud.tencent.com/developer/article/2051064)，这个可能只是为了开发规范和管理，从技术角度出发，其实是没有这样的原则性问题的。

解决方案：使用SplitQuery，从字面意义就可以理解，即将这些join查询拆分成单个查询来执行

示例代码（推荐）：

```sql
var data = ctx.As
    .Include(x => x.Bs)
    .Include(x => x.Cs)
    .ThenInclude(x => x.D1s)
    .Include(x => x.Cs)
    .ThenIncude(x => x.C1s)
    .ThenInclude(x=>x.D2s)
    .AsSplitQuery() //设置为拆分查询
    .ToList();
```

当然也可以在全局进行配置 (但是一般不推荐这样做，最好根据每个查询的实际情况，使用上面推荐的方式)

```csharp
builder.Services.AddDbContext<CRSGEntityDbContext>(options => options.UseSqlServer(builder.Configuration["ConnectionStrings:FiinGroupDB"], o => o.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery)));
```

生成的sql

```sql
SELECT [a].[Id], [a].[OtherColumns]
FROM [As] AS [a]

SELECT [b].[Id], [b].[AId], [b].[OtherColumns]
FROM [Bs] AS [b]
INNER JOIN [As] AS [a] ON [b].[AId] = [a].[Id]

SELECT [c].[Id], [c].[AId], [c].[OtherColumns]
FROM [Cs] AS [c]
INNER JOIN [As] AS [a] ON [c].[AId] = [a].[Id]

SELECT [d1].[Id], [d1].[CId], [d1].[OtherColumns]
FROM [D1s] AS [d1]
INNER JOIN [Cs] AS [c] ON [d1].[CId] = [c].[Id]
WHERE [c].[AId] IN (SELECT [a].[Id] FROM [As] AS [a])

SELECT [c1].[Id], [c1].[CId], [c1].[OtherColumns]
FROM [C1s] AS [c1]
INNER JOIN [Cs] AS [c] ON [c1].[CId] = [c].[Id]
WHERE [c].[AId] IN (SELECT [a].[Id] FROM [As] AS [a])

SELECT [d2].[Id], [d2].[C1Id], [d2].[OtherColumns]
FROM [D2s] AS [d2]
INNER JOIN [C1s] AS [c1] ON [d2].[C1Id] = [c1].[Id]
WHERE [c1].[CId] IN (SELECT [c].[Id] FROM [Cs] AS [c] WHERE [c].[AId] IN (SELECT [a].[Id] FROM [As] AS [a]))
```

可以看到查询被拆分成了独立的语句，逻辑更加清晰，对于数据库来说执行效率也会更好。

> 注意事项：虽然拆分查询可以通过避免笛尔卡爆炸带来的性能问题，但是也需要根据实际的查询场景来决定是否使用，例如，需要对数据进行排序，分页，分组等操作的时候，为了保证查询结果的正确性，就需要考虑是否要使用拆分查询

关联话题：关于懒加载，其实懒加载的问题原因就等同于在循环中执行sql语句，示例代码：

```csharp
//在没有显示加载的情况下，直接循环查询子对象
foreach (var blog in context.Blogs.ToList())
{
    foreach (var post in blog.Posts)
    {
        Console.WriteLine($"Blog {blog.Url}, Post: {post.Title}");
    }
}
```

观察sql日志：

```
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT [b].[BlogId], [b].[Rating], [b].[Url]
      FROM [Blogs] AS [b]
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (5ms) [Parameters=[@__p_0='1'], CommandType='Text', CommandTimeout='30']
      SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Title]
      FROM [Post] AS [p]
      WHERE [p].[BlogId] = @__p_0
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (1ms) [Parameters=[@__p_0='2'], CommandType='Text', CommandTimeout='30']
      SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Title]
      FROM [Post] AS [p]
      WHERE [p].[BlogId] = @__p_0
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (1ms) [Parameters=[@__p_0='3'], CommandType='Text', CommandTimeout='30']
      SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Title]
      FROM [Post] AS [p]
      WHERE [p].[BlogId] = @__p_0
... and so on
```

正确的做法即使用Include或者Load显示加载数据。

## 使用批处理语句

批处理语句是EFCore7 版本中更新的重要功能，解决了以往版本需要借助第三方库来实现数据的批量更新，删除操作，而且在性能上带来了更大的提升

### 批量删除

之前版本的做法（不借助第三方库）

```csharp
foreach (var blog in context.Blogs.Where(b => b.Rating < 3))
{
    context.Blogs.Remove(blog);
}
context.SaveChanges();
```

使用ExecuteDelete，无论是从语法上还是性能上，批处理操作都优于前者。

```
context.Blogs.Where(b => b.Rating < 3).ExecuteDelete();
```

如果是EFCore版本低于7.0，也可以使用直接执行sql语句 ExecuteSqlRaw 的方式来进行操作

```
context.Database.ExecuteSqlRaw("DELETE FROM [Blogs] WHERE [Rating] < 3");
```

### 批量更新

用法与Delete 基本相同

```
context.Blogs
    .Where(b => b.Rating < 3)
    .ExecuteUpdate(setters => setters.SetProperty(b => b.IsVisible, false));
```

> 注意事项：目前仅支持关系型数据库，而且需要由于是及时发送上下文请求，所以如果要支持事务，需要使用显示事务来与其他代码组合

## 使用非跟踪查询

这个比较简单，在你不需要对查询结果进行任何更新操作的场景下，尽量使用非跟踪查询

```
var blogs = context.Blogs
    .AsNoTracking()
    .ToList();
```

或者

```
context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
var blogs = context.Blogs.ToList();
```

测试代码

```
//代码执行前已对数据库进行预热处理
//执行5次
double elapsedTime4 = MeasureTime(() => context.Blogs.FirstOrDefault(x => x.Id == 1), 5);
double elapsedTime5 = MeasureTime(() => context.Blogs.AsNoTracking().FirstOrDefault(x => x.Id == 1), 5);

Console.WriteLine($"Tracked time took : {elapsedTime4} ms");
Console.WriteLine($"AsNoTracking() time took : {elapsedTime5} ms");

//Consoles:
//Tracked time took : 318.26 ms
//AsNoTracking() time took : 229.86 ms
```

## 仅投影需要的字段

严格意义上来讲这是一个意识问题，大多数情况下，为了节省代码量，可以直接使用DataSet 定义的对象来直接进行查询，或者使用Include加载关联表数据，但是在遇到大量数据查询或大量的表连接查询的时候，精准的属性投影对性能就会起到很大的影响

示例代码：

```
var data = ctx.As
    .Where(x => x.Name.StartWith("xxx"))
    .ToList();

foreach (var item in data.Bs)
{
    Console.WriteLine($"Name :{item.Name},Id: {item.Id}");
}
```

上述代码中，我们仅需要查询主表及子表的id和name信息，但是却加载了所有的相关的主表和子表字段，这对性能是一种浪费

解决方案：通过Select投影需要查询的字段

```
var data = ctx.As
    .Where(x => x.Id=1)
    .Select(x => new {x.Id, x.Name})
    .ToList();
```

个人习惯性做法

```
//1，不依赖于数据库外键的设置
var query = from b in context.Blogs
            join c in context.Comments on b.blogId equals c.blogId
            join d in context.Posts on d.commentId equals c.commentId
            select new A{blogId = b.blogId,postId = d.postId,postValue = d.postValue}
```

> 这种做法对多表联查和大数据量的查询很有用 ,但需要注意的是这种做法并不适合需要更新数据的场景，因为 EF 的更改跟踪仅适用于实体实例。

## 尽量使用异步方法

EFCore 基本上对所有同步操作方法都提供了对应的异步方法，尽量使用他们避免阻塞，减少对线程的需要和必须发生的线程上下文切换的次数，从而提升性能。

```
//ToListAsync
var data = await context.blogs.ToListAsync();
//FirstOrDefaultAsync
var item = await context.blogs.FirstOrDefaultAsync(it => it.Id == 1);
item.point=2;
//SaveChangesAsync
await context.SaveChangesAsync();
//AsAsyncEnumerable
var groupedHighlyRatedBlogs = await context.Blogs
    .AsQueryable()
    .Where(b => b.Rating > 3) // server-evaluated
    .AsAsyncEnumerable()
    .GroupBy(b => b.Rating) // client-evaluated
    .ToListAsync();
```

> 异步编程在efcore中在大多数情况被推荐使用，但是需要注意避免使用异步方法查询文本或二进制数据类型的内容，这样反而会引起性能问题([sqlclient](https://github.com/dotnet/SqlClient)的问题)，issue报告: [EF Core - Memory and performance issues with async methods](https://github.com/dotnet/efcore/issues/21147) [Reading large data (binary, text) asynchronously is extremely slow](https://github.com/dotnet/SqlClient/issues/593)

> 避免混合使用同步和异步方法，当你的程序请求量较大的时候，很可能导致连接池耗尽，从而引起的性能问题。

## 使用Find查找单个目标数据

设计为在已知主键时高效查找单个实体。 Find 首先检查实体是否已被跟踪，如果是，则立即返回该实体。 只有当未在本地跟踪实体时，才执行数据库查询，而First/FirstOrDefault会立即查询数据库。

```
//代码执行前已对数据库进行预热处理
double elapsedTime4 = MeasureTime(() => context.blogs.Find(1);
double elapsedTime5 = MeasureTime(() => context.blogs.Find(1);

Console.WriteLine($"Find() first time took : {elapsedTime4} ms");
Console.WriteLine($"Find() second time took : {elapsedTime5} ms");

//Consoles:
//Find() first time took : 268.41 ms
//Find() second time took : 0.16 ms
```

> 注意，只能通过键查询的时候可以用。

## 使用Any判断数据内容

在检查某些数据是否存在的时候，优先使用Any，这样在匹配到第一条数据后，查询就会停止，First因为需要返回数据，增加了数据传输和对象实例化的开销，Count则需要扫描表

```
double elapsedTime1 = MeasureTime(() => context.Blogs.Any(it => it.Id == 1));
double elapsedTime2 = MeasureTime(() => context.Blogs.Count(it => it.Id == 1), 1);
double elapsedTime3 = MeasureTime(() => context.Blogs.FirstOrDefault(it => it.Id == 1), 1);

Console.WriteLine($"Any() time took: {elapsedTime1} ms");
Console.WriteLine($"Count() time took: {elapsedTime2} ms");
Console.WriteLine($"FirstOrDefault() time took: {elapsedTime3} ms");

//Consoles:
//Any() time took: 237.42 ms
//Count() time took: 239.69 ms
//FirstOrDefault() time took: 258.28 ms
```

## 使用流式处理

首先了解什么是缓冲和流式处理

- 缓冲：将需要的数据全部加载到内存中，用于后续的业务逻辑处理
- 流式处理：按需获取需要的数据并应用到后续的逻辑处理中

形象的理解，缓冲用水桶把水挑起来，然后倒进缸里，流式处理就是用一根水管把水抽到缸里

原则上，流式处理查询的内存要求是固定的：无论查询返回 1 行还是 1000 行，内存要求都相同。另一方面，返回的行数越多，缓冲查询需要的内存越多。 对于产生大型结果集的查询，这可能是一个重要的性能因素。 反之，如果你的查询结果量很小，那么使用缓冲的效果可能返回会更好。

```
//一次性将数据加载出来
var blogsList = context.Posts.Where(p => p.Title.StartsWith("A")).ToList();
var blogsArray = context.Posts.Where(p => p.Title.StartsWith("A")).ToArray();

//使用流式处理，每次处理一行
foreach (var blog in context.Posts.Where(p => p.Title.StartsWith("A")))
{
    //do some things...
    SomeDotNetMethod(blog)
}

// 也可以使用AsEnumerable实现
var doubleFilteredBlogs = context.Posts
    .Where(p => p.Title.StartsWith("A")) // 执行数据库查询
    .AsEnumerable()
    .Where(p => SomeDotNetMethod(p)); //执行客户端操作
```

> 流式处理适合处理大量数据需要进行某些业务逻辑的加工或执行，但是数据库又无法支持响应的方法或函数，这个时候可以适用流式处理来进行操作。

## 使用SQL查询

在某些特殊的情况下，例如一些复杂的sql查询，无法直接使用linq语法来实现的，EFCore也支持直接使用SQL语句进行查询或数据更新操作

### 基本查询(实体)

场景：最终返回的结果与Dataset中定义的实体一致

```
//使用FromSql

//执行表查询
var blogs = context.Blogs
    .FromSql($"SELECT * FROM dbo.Blogs")
    .ToList();

//执行存储过程查询返回实体
var blogs = context.Blogs
    .FromSql($"EXECUTE dbo.GetMostPopularBlogs")
    .ToList();
```

### 标量查询(非实体)

场景：最终返回的结果为自定义结构，而非数据库实体

```csharp
//使用SqlQuery

//执行查询，返回单个字段
var ids = context.Database
    .SqlQuery<int>($"SELECT [BlogId] FROM [Blogs]")
    .ToList();

//执行查询，返回自定义数据结构
var comments = context.Database
    .SqlQuery<int>($"SELECT b.[BlogId],c.[CommnetContent] FROM [Blogs] b JOIN [Comments] c on b.BlogId = c.BlogId")
    .ToList();

public class CustomBlog{
    public int BlogId
    public string CommnetContent
}
```

### 执行非查询SQL

场景：提交更新，删除等操作，不关注返回结果

```
//使用ExecuteSql

//执行更新
context.Database.ExecuteSql($"UPDATE [Blogs] SET [Url] = NULL WHERE Id =1");
//执行删除
context.Database.ExecuteSql($"DELETE FROM [Blogs] WHERE Id =1");
```

### SQL参数

```csharp
//使用FromSql

//此代码无效，因为数据库不允许将列名（或架构的任何其他部分）参数化
var propertyName = "User";
var propertyValue = "johndoe";

var blogs = context.Blogs
    .FromSql($"SELECT * FROM [Blogs] WHERE {propertyName} = {propertyValue}")
    .ToList();

//正确姿势：使用 FromSqlRaw
var columnName = "Url";
var columnValue = new SqlParameter("columnValue", "http://SomeURL");

var blogs = context.Blogs
    .FromSqlRaw($"SELECT * FROM [Blogs] WHERE {columnName} = @columnValue", columnValue)
    .ToList();
```

## 其他关联性优化

除了针对EFCore本身的一些优化技巧之外，还有一些技巧可以帮助我们提升数据查询的效率,我们可以利用vs的调试工具帮助我们监听内存使用,CPU占用率等指标，查找瓶颈，总结主要从以下几个方面进行优化

1.  尽量避免循环内查询，分析实际的业务逻辑，尽可能的一次性从数据库加载所有需要的数据，再进行循环处理
2.  分片处理条件数据,例如使用Chunk，使用流式处理大批量的数据集的运算
3.  使用合理的数据结构，例如在不关注数据顺序的场景下使用Dictionary或HashSet代替List等
4.  使用缓存减少热点数据的访问（按需设计）
5.  使用数据表索引及物化视图（数据库）
6.  采用分库分表，读写分离，使用ES进行检索（架构级优化）
7.  利用多线程并发提升效率（不到万不得已，慎用）

## 总结

EFCore的优化主要是从几个方面来进行：  
1.减少数据库的交互，通过连接复用，上下文缓存等  
2.减少内存的使用，例如使用流式处理，分页查询等  
3.降低查询复杂度，尽量在程序中处理复杂的逻辑

保持良好的编码习惯，使用正确的数据结构和处理逻辑，优化应该是渐进式的，先正确的满足需求，在遇到性能问题的时候借助代码或工具去分析瓶颈，再去进行针对性的优化，不要为了优化而牺牲需求和浪费工作量。

最后留给大家一段问题代码示例，感兴趣的童鞋可以尝试利用上述手段优化这段代码，看看效率提升有多少：

```csharp
var configuration = new ConfigurationBuilder()
            .SetBasePath(AppDomain.CurrentDomain.BaseDirectory)
            .AddUserSecrets<Program>()
            .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
            .Build();

var serviceProvider = new ServiceCollection()
            .AddDbContext<YouContext>(options =>
                options.UseSqlServer(configuration["ConnectionStrings:YourContext"])
                        .EnableSensitiveDataLogging()
                        .UseLoggerFactory(LoggerFactory.Create(builder =>
                            {
                                builder.AddConsole().AddFilter((category, level) => category == DbLoggerCategory.Database.Command.Name && level == LogLevel.Information);
                            })))
            .BuildServiceProvider();

using (var scope = serviceProvider.CreateScope())
{
var context = scope.ServiceProvider.GetRequiredService<YourContext>();
context.Database.SetCommandTimeout(999);
    var data = context.RELATIONSHIP.Where(x => x.workflow_state == 3).OrderBy(it => it.relationship_guid).Take(100000).ToList();
    var tempData = data.Select(it => new Temp { aId = it.entity_from_guid, bId = it.entity_to_guid, deg = 0 }).ToList();

    foreach (var item in tempData)
    {
        item.deg = GetInterConnectResult(item.aId, item.bId);
    }

    int GetInterConnectResult(Guid aId, Guid bId)
    {
        HashSet<Bo> boData = new();
        for (int i = 1; i <= 3; i++)
        {
            if (boData.Any(it => it.guid == bId)) break;
            var addIds = boData.Where(it => it.deg == i - 1).Select(it => it.guid).Distinct().ToList();
            var addRelationships = context.Table1.Where(it => addIds.Contains(it.aid) || addIds.Contains(it.bid));

            var addDegEntities = addRelationships.Select(it => new
            {
                efguid = it.aid,
                etguid = it.bid
            }).Union(addRelationships.Select(it => new
            {
                efguid = it.bid,
                etguid = it.aid
            }))
            .Select(it => new Bo { guid = it.etguid, deg = i })
            .ToHashSet() ?? new();
            boData.UnionWith(addDegEntities ?? new());
        }

        return boData?.OrderByDescending(it => it.deg)?.FirstOrDefault(it => it.guid.Equals(bId))?.deg ?? 0;
    }
}
```
