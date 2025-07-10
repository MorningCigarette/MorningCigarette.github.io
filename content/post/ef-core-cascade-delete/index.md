---
title: "数据库级联限制"
description: 
slug: "ef-core-cascade-delete"
date: 2025-07-10
categories:
    - EFCORE

---

某些数据库（尤其是 SQL Server）对形成周期的级联行为有限制。 例如，请考虑以下模型：

```
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; }

    public IList<Post> Posts { get; } = new List<Post>();

    public int OwnerId { get; set; }
    public Person Owner { get; set; }
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; }
    public Blog Blog { get; set; }

    public int AuthorId { get; set; }
    public Person Author { get; set; }
}

public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }

    public IList<Post> Posts { get; } = new List<Post>();

    public Blog OwnedBlog { get; set; }
}
```

此模型具有三种关系，全部必需，因此配置为按约定级联删除：

- 删除博客将同时删除所有相关帖子
- 删除帖子的作者将导致其创作的帖子被级联删除。
- 删除博客所有者将导致博客被级联删除

这一切都是合理的（虽然博客管理策略有些严厉！），但尝试配置这些级联来创建 SQL Server 数据库会引发以下异常：

> Microsoft.Data.SqlClient.SqlException （0x80131904）：在表“Post”上引入 FOREIGN KEY 约束“FK_Posts_Person_AuthorId”可能会导致循环或多个级联路径。 请指定 ON DELETE NO ACTION 或 ON UPDATE NO ACTION，或修改其他 FOREIGN KEY 约束。

有两种方法可以处理这种情况：

1. 更改一个或多个关系以使其不进行级联删除。
2. 在没有一个或多个此类级联删除的情况下配置数据库，然后确保加载所有依赖实体，以便 EF Core 可以执行级联行为。

通过示例采用第一种方法，我们可以通过为其提供可为 null 的外键属性来使博客后的关系可选：

 

```
public int? BlogId { get; set; }
```

可选关系允许帖子在没有博客的情况下存在，这意味着默认情况下将不再配置级联删除。 这意味着在级联作中不再存在循环，可以在 SQL Server 上创建数据库而不出错。

改用第二种方法，我们可以保留博客与所有者的关系，并配置为级联删除，但使此配置仅适用于被跟踪的实体，而不是整个数据库。

 

```
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .HasOne(e => e.Owner)
        .WithOne(e => e.OwnedBlog)
        .OnDelete(DeleteBehavior.ClientCascade);
}
```

现在，如果我们同时加载一个人和他们拥有的博客，那么会发生什么情况，然后删除该人员？

 

```
using var context = new BlogsContext();

var owner = await context.People.SingleAsync(e => e.Name == "ajcvickers");
var blog = await context.Blogs.SingleAsync(e => e.Owner == owner);

context.Remove(owner);

await context.SaveChangesAsync();
```

EF Core 将级联删除所有者，以便同时删除博客：

 

```
-- Executed DbCommand (8ms) [Parameters=[@p0='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Blogs]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;

-- Executed DbCommand (2ms) [Parameters=[@p1='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [People]
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;
```

但是，如果在删除所有者时博客未加载：

```
using var context = new BlogsContext();

var owner = await context.People.SingleAsync(e => e.Name == "ajcvickers");

context.Remove(owner);

await context.SaveChangesAsync();
```

然后，由于违反数据库中的外键约束，将引发异常：

> Microsoft.Data.SqlClient.SqlException：DELETE 语句与 REFERENCE 约束“FK_Blogs_People_OwnerId”冲突。 数据库“Scratch”中表“dbo.Blogs”的列“OwnerId”发生冲突。 语句已终止。
