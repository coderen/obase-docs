# 实体类型配置

<a name="LPIxG"></a>
#### 涉及Obase对象
![](https://cdn.nlark.com/yuque/__puml/218a401e64d89e78622f8697a0f7003e.svg#lake_card_v2=eyJjb2RlIjoiY2xhc3MgXCJPYmplY3RDb250ZXh0W-WvueixoeS4iuS4i-aWh11cIiBhcyBPYmplY3RDb250ZXh0e1xuXHQjT25Nb2RlbENyZWF0aW5nKG1vZGVsQnVpbGRlcilcbn1cblxuY2xhc3MgXCJNb2RlbEJ1aWxkZXJb5bu65qih5ZmoXVwiIGFzIE1vZGVsQnVpbGRlcntcblx0K0VudGl0eTxURW50aXR5PigpOkVudGl0eVR5cGVDb25maWd1cmF0aW9uPFRFbnRpdHk-XG59XG5cbmNsYXNzIFwiRW50aXR5VHlwZUNvbmZpZ3VyYXRpb248VEVudGl0eT5b5a6e5L2T5Z6L6YWN572u6aG5XVwiIGFzIEVudGl0eVR5cGVDb25maWd1cmF0aW9uIHtcblx0K1RvVGFibGUodGFibGUpXG5cdCtIYXNLZXlBdHRyaWJ1dGUoZXhwcmVzc2lvbilcblx0K0hhc0tleUlzU2VsZkluY3JlYXNlZChrZXlJc1NlbGZJbmNyZWFzZWQpXG5cdCtIYXNWZXJzaW9uQXR0cmlidXRlKGV4cHJlc3Npb24pXG5cdCtBdHRyaWJ1dGUoZXhwcmVzc2lvbilcblx0K0lnbm9yZShleHByZXNzaW9uKVxuXHQrSGFzVmVyc2lvbkF0dHJpYnV0ZShleHByZXNzaW9uKVxuXHQrSGFzQ29uY3VycmVudENvbmZsaWN0SGFuZGxpbmdTdHJhdGVneShlQ29uY3VycmVudENvbmZsaWN0SGFuZGxpbmdTdHJhdGVneSlcblx0K0Fzc29jaWF0aW9uUmVmZXJlbmNlPFRFbmQxLCBURW5kMj4obmFtZSxpc011bHRpcGxlKVxuXHQrQXNzb2NpYXRpb25SZWZlcmVuY2U8VEFzc29jaWF0aW9uPihuYW1lLGlzTXVsdGlwbGUpIFxuXHQrQXNzb2NpYXRpb25SZWZlcmVuY2U8VFJlc3VsdD4oZXhwcmVzc2lvbilcbn1cblxuT2JqZWN0Q29udGV4dCAtcmlnaHQtIE1vZGVsQnVpbGRlclxuTW9kZWxCdWlsZGVyIC0tIEVudGl0eVR5cGVDb25maWd1cmF0aW9uXG5cbmhpZGUgZW1wdHkgbWVtYmVyIiwidHlwZSI6InB1bWwiLCJtYXJnaW4iOnRydWUsImlkIjoiZkpvd0oiLCJ1cmwiOiJodHRwczovL2Nkbi5ubGFyay5jb20veXVxdWUvX19wdW1sLzIxOGE0MDFlNjRkODllNzg2MjJmODY5N2EwZjcwMDNlLnN2ZyIsImNhcmQiOiJkaWFncmFtIn0=)<a name="atPRR"></a>
#### 
<a name="xEb3h"></a>
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
<a name="nMZc7"></a>
#### 
<a name="NDAb7"></a>
#### 映射数据表
实体映射到具体的数据库表，实现方式如下：
```csharp
blogCfg.ToTable("Blog");
```
<a name="zVACn"></a>
#### 
<a name="WGUvj"></a>
#### 设置标识属性
根据 Lamda 表达式包含的信息设置标识属性，每调用一次本方法，追加一个标识属性：
```csharp
blogCfg.HasKeyAttribute(p => p.BlogId);
```
<a name="R91tR"></a>
#### 
<a name="nWm34"></a>
#### 标识属性自增长
如果标识属性在数据库中为自增长的时候，需要通过以下方式进行设置：
```csharp
blogCfg.HasKeyIsSelfIncreased(true);
```
<a name="soxIu"></a>
#### 
<a name="QzV5N"></a>
#### 属性映射表字段
默认情况下实体的属性会映射到与属性同名的表字段，如果希望映射不同的字段可以通过以下方式实现：
```csharp
blogCfg.Attribute(p => p.Url).ToField("m_Url");
```
<a name="nQGp9"></a>
#### 
<a name="GHrOf"></a>
#### 从模型中排除某个属性
当某个属性不需要配置到模型中的时候，可以告知建模器不为该属性生成类型元素配置：
```csharp
blogCfg.Ignore(p => p.Url);
```
<a name="rnw0D"></a>
#### 
<a name="r5fhT"></a>
#### 设置版本标识属性集
使用 Obase 的时候如果要实现乐观锁并发控制，可以使用 Version 属性完成版本管理，具体如何设置 Version 属性如下：
```csharp
blogCfg.HasVersionAttribute(p => p.Version);
```
HasVersionAttribute 方法每调用一次将会为实体追加一个版本标识属性。
<a name="h2WWN"></a>
#### 
<a name="nCgmK"></a>
#### 设置版本冲突处理策略
当出现版本冲突时，即 Obase 中的对象 Version 属性与数据库中 Version 字段不匹配的时候，需要设置版本冲突的处理策略，例如：忽略、强制覆盖、版本合并、重建对象、抛出异常（默认），具体设置方式如下：
```csharp
 blogCfg.HasConcurrentConflictHandlingStrategy(eConcurrentConflictHandlingStrategy.Combine);
```
<a name="7wlBR"></a>
#### 配置实体的关联引用
当实体与其它实体有关联的时候，需要配置实体的关联引用属性，配置方式如下：
```csharp
var blogRefPosts = blogCfg.AssociationReference<Blog, Post>("Posts", true);
```
此处即在配置 Blog 实体与 Post 实体的关联引用属性 Posts ，AssociationReference 方法有3个重载，分别可以用于关联 映射关联型和显示关联型。关于关联应用配置此处只做简单的了解，有关详细信息，请参阅[关联引用配置](https://www.yuque.com/obase/help/associationreference)。
