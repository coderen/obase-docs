# 实体属性配置
>当通过模型建模器的 `Entity()` 方法获取到实体型配置之后，可以通过实体型配置的`Attibute()`方法获取到实体的属性配置对象，在实体属性配置对象上我们可以完成对属性 映射字段、取值器、设值器、触发器 等的配置

---
#### 获取实体属性配置
Attribute 方法获取到的实体属性配置为 AttributeConfiguration<T> 类型，在此类型的实例化对象上我们可以完成我们自己的配置，当然大多数情况下建议直接采用链式书写法直接在`Attibute()`方法后完成具体方法的调用：
```csharp
AttributeConfiguration<Blog> urlAttributeConfiguration = blogCfg.Attribute(p => p.Url);
```
---
#### 属性映射表字段
默认情况下实体的属性会映射到与属性同名的表字段，如果希望映射不同的字段可以通过以下方式实现：
```csharp
blogCfg.Attribute(p => p.Url).ToField("m_Url");
```

---
#### 属性取值器与设值器
>属性取值器与设值器主要是用来配置 Obase 获取或者设置对象属性值的方式。当 Obase 需要读取实体型对象的属性值时（例如：来完成持久化的时候）需要使用属性配置的取值器，当 Obase 需要赋值给实体型对象时（例如：将数据库表数据反持久化到实体型对象）需要使用属性配置的设值器。Obase 默认自动为属性配置的取值器和设值器为：封装了属性自身的 `get` 访问器与 `set` 访问器的委托取值器和设值器。

##### 自定义属性取值器
如果需要自定义属性的取值器，可以通过属性配置的 `HasValueGetter()`方法完成。例如此处`Blog`实体`List<string>`类型的`Tags`属性需要存储到数据库的时候，就需要转换为`string`类型，需要做类型转换，所以需要自定义属性取值器。具体方法如下
```plantuml
class Blog{
    +Tags:List<string>[数据库中为varchar类型]
}
hide empty member
```
```csharp
blogCfg.Attribute(p => p.Tags).HasValueGetter<string>(p =>
{
    return string.Join(",", p?.Tags);
})
```

##### 自定义属性设值器
如果需要自定义属性的取值器，可以通过属性配置的 `HasValueSetter()`方法完成。例如`Blog`的`Tags`属性在数据库中为`string`类型，取出数据赋值给`List<string>`类型的`Tags`属性时，需要做类型转换，所以就需要自定义属性设置器。具体方法如下：
```csharp
blogCfg.Attribute(p => p.Tags).HasValueSetter<string>((a, v) =>
{
    a.Tags = new List<string>(v.Split(","));
});
```