# 实体类型配置

#### 涉及Obase对象
```plantuml
class "ObjectContext[对象上下文]" as ObjectContext{
	#OnModelCreating(modelBuilder)
}

class "ModelBuilder[建模器]" as ModelBuilder{
	+Entity<TEntity>():EntityTypeConfiguration<TEntity>
}

class "EntityTypeConfiguration<TEntity>[实体型配置项]" as EntityTypeConfiguration {
	+ToTable(table)
	+HasKeyAttribute(expression)
	+HasKeyIsSelfIncreased(keyIsSelfIncreased)
	+HasVersionAttribute(expression)
	+Attribute(expression)
	+Ignore(expression)
	+HasVersionAttribute(expression)
	+HasConcurrentConflictHandlingStrategy(eConcurrentConflictHandlingStrategy)
	+AssociationReference<TEnd1, TEnd2>(name,isMultiple)
	+AssociationReference<TAssociation>(name,isMultiple) 
	+AssociationReference<TResult>(expression)
}

ObjectContext -right- ModelBuilder
ModelBuilder -- EntityTypeConfiguration

hide empty member
```

---
#### 配置位置
在自定义的 ObjectContext 的子类中，重写 OnModelCreating 方法，在此方法中调用 ModelBuilder 的 Entity() 方法得到表示实体型配置的 EntityTypeConfiguration 对象，此对象中包含实体的所有配置方法，具体配置位置如下：
```csharp
public class SimpleContext : ObjectContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        var blogCfg = modelBuilder.Entity<Blog>();
    }
}
```

---
#### 映射数据表
实体映射到具体的数据库表，实现方式如下：
```csharp
blogCfg.ToTable("Blog");
```

---
#### 设置标识属性
根据 Lamda 表达式包含的信息设置标识属性，每调用一次本方法，追加一个标识属性：
```csharp
blogCfg.HasKeyAttribute(p => p.BlogId);
```

---
#### 标识属性自增长
如果标识属性在数据库中为自增长的时候，需要通过以下方式进行设置：
```csharp
blogCfg.HasKeyIsSelfIncreased(true);
```

---
#### 从模型中排除某个属性
当某个属性不需要配置到模型中的时候，可以告知建模器不为该属性生成类型元素配置：
```csharp
blogCfg.Ignore(p => p.Url);
```

---
#### 设置版本标识属性集
使用 Obase 的时候如果要实现乐观锁并发控制，可以使用 Version 属性完成版本管理，具体如何设置 Version 属性如下：
```csharp
blogCfg.HasVersionAttribute(p => p.Version);
```
HasVersionAttribute 方法每调用一次将会为实体追加一个版本标识属性。

---
#### 设置版本冲突处理策略
当出现版本冲突时，即 Obase 中的对象 Version 属性与数据库中 Version 字段不匹配的时候，需要设置版本冲突的处理策略，例如：忽略、强制覆盖、版本合并、重建对象、抛出异常（默认），具体设置方式如下：
```csharp
 blogCfg.HasConcurrentConflictHandlingStrategy(eConcurrentConflictHandlingStrategy.Combine);
```

---
#### 配置实体的关联引用
当实体与其它实体有关联的时候，需要配置实体的关联引用属性，配置方式如下：
```csharp
var blogRefPosts = blogCfg.AssociationReference<Blog, Post>("Posts", true);
```
此处即在配置 Blog 实体与 Post 实体的关联引用属性 Posts ，AssociationReference 方法有3个重载，分别可以用于关联 映射关联型和显示关联型。关于关联应用配置此处只做简单的了解，有关详细信息，请参阅[关联引用配置](association-reference-configuration.md)。
