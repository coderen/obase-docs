# Obase快速入门

在项目中完成对象建模后，可以使用Obase来进行对象的管理（例如对象持久化），本篇教程将创建一个.NET Core控制台应用，来展示Obase的配置和对象的增删改查操作。本篇教程旨在指引简单入门.<br />
<br />本篇教程将以此对象模型展开<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/291445/1580630239143-cfab9c29-f8a8-4def-b99c-a859d3bb9fed.png#align=left&display=inline&height=251&margin=%5Bobject%20Object%5D&name=image.png&originHeight=204&originWidth=400&size=15575&status=done&style=none&width=493)<br />

<a name="coGkZ"></a>
### 1. 从NuGet安装Obase

- 在VS的NuGet包管理器中添加程序包源：[http://nuget.suiyiyun.cn:8081/nuget](http://nuget.suiyiyun.cn:8081/nuget)
- 在NuGet包管理器中选择Obase进行安装


<br />

<a name="1eYCn"></a>
### 2. 项目搭建

- 打开 Visual Studio
- 单击“创建新项目”
- 选择带有 C# 标记的“控制台应用 (.NET Core)” ，然后单击“下一步”
- 输入“ObaseTutorial” 作为名称，然后单击“创建”
- 添加对freep.Obase.dll的引用



<a name="ENXfm"></a>
### 3. 定义领域实体类
```csharp
	/// <summary>
    /// 文章
    /// </summary>
    public class Blog
    {
        private int blogId;
        private string url;
        private List<Post> posts;

        /// <summary>
        /// 文章Id
        /// </summary>
        public int BlogId { get => blogId; set => blogId = value; }
        /// <summary>
        /// 文章地址
        /// </summary>
        public string Url { get => url; set => url = value; }
       /// <summary>
       /// 文章评论（注意：关联引用属性需要定义为virtual）
       /// </summary>
        public virtual List<Post> Posts { get => posts; set => posts = value; }
    }

   /// <summary>
    /// 文章评论
    /// </summary>
    public class Post
    {
        private int postId;
        private string title;
        private string content;
        private int blogId;
        private Blog blog;

        /// <summary>
        /// 评论Id
        /// </summary>
        public int PostId { get => postId; set => postId = value; }
        /// <summary>
        /// 评论标题
        /// </summary>
        public string Title { get => title; set => title = value; }
        /// <summary>
        /// 评论内容
        /// </summary>
        public string Content { get => content; set => content = value; }
        /// <summary>
        /// 文章Id
        /// </summary>
        public int BlogId { get => blogId; set => blogId = value; }
        /// <summary>
        /// 关联文章（注意：关联引用属性需要定义为virtual）
        /// </summary>
        public virtual Blog Blog { get => blog; set => blog = value; }
    }
```


<a name="aodnK"></a>
### 4. 自定义对象上下文
Obase直接与应用程序进行交互的便是ObectContext（对象上下文），项目中可以根据具体情况定义一个或者多个继承于ObjectContext的自定义对象上下文。
```csharp
	using freep.Obase;
	using freep.Obase.ExecuteSql;
	using freep.Obase.Odm.Builder;

	/// <summary>
    /// 自定义对象上下文
    /// </summary>
    public class MyContext : ObjectContext
    {
        /// <summary>
        /// 构造函数
        /// </summary>
        public MyContext() : base("user=root;password=;server=localhost;database=ObaseTutorial;SslMode = none;port=3306;", true)
        {
        }
    }
```
注意：自定义对象上下文通过继承父类的构造函数设置数据源连接字符串（此处为了演示方便，直接将连接字符串作为参数进行传递，实际项目中可以定义到配置文件中）。<br />

<a name="aXFTv"></a>
### 5. 配置对象模型
在对象数据模型生成之前，可以对数据源的类型进行设置，以及对象数据模型的配置，配置的类型包括实体类型，关联类型，关联引用，关联端，属性等的配置，本篇只展示最基本的实体类型，关联类型，关联引用的配置。
```csharp
    /// <summary>
    /// 自定义对象上下文
    /// </summary>
    public class MyContext : ObjectContext
    {
		/// <summary>
        /// 在即将生成对象数据模型并注册到对象上下文之前调用此方法
        /// </summary>
        /// <param name="modelBuilder">建模器</param>
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            //设置模型映射目标源的类型(默认不设置未SQLServer)
            modelBuilder.HasTargetSourceType(eDataSource.MySql);
            //配置对象数据模型
            this.ModelConfiguratoin(modelBuilder);
            base.OnModelCreating(modelBuilder);
        }
        
        /// <summary>
        /// 配置对象数据模型
        /// </summary>
        /// <param name="modelBuilder">建模器</param>
        protected virtual void ModelConfiguratoin(ModelBuilder modelBuilder)
        {
        	//配置实体型
            var blogCfg = modelBuilder.Entity<Blog>();
            //设置实体型的映射数据表
            blogCfg.ToTable("Blog");
            //设置实体型的标识属性
            blogCfg.HasKeyAttribute(p => p.BlogId);
            //设置实体型的标识属性为自增
            blogCfg.HasKeyIsSelfIncreased(true);

            //配置实体型
            var postCfg = modelBuilder.Entity<Post>();
            //设置实体型的映射数据表
            postCfg.ToTable("Post");
            //设置实体型的标识属性
            postCfg.HasKeyAttribute(p => p.PostId);
            //设置实体型的标识属性为自增
            postCfg.HasKeyIsSelfIncreased(true);

            //配置对象间隐式关联类型
            var blogAssPostCfg = modelBuilder.Association<Blog, Post>();
            //设置关联类型的映射数据表
            blogAssPostCfg.ToTable("Post");
            //设置关联映射端1（参照方）的键属性以及在关联表中映射的字段
            blogAssPostCfg.AssociationEnd<Blog>("End1").HasMapping("BlogId", "BlogId");
            //设置关联映射端2（被参照方）的键属性以及在关联表中映射的字段
            //注意：HasDefaultAsNew方法设置一个值，该值指示是否把关联端对象默认视为新对象。当该属性为true时，如果关联端对象未被显式附加到上下文，该对象将被视为新对象实施持久化。
            blogAssPostCfg.AssociationEnd<Post>("End2").HasMapping("PostId", "PostId").HasDefaultAsNew(true);

            //配置实体类型的关联引用属性
            //参数一：关联引用属性的名称 参数二：关联引用是否具有多重性
            //注：此处在配置Blog实体与Post实体关联引用属性Posts
            var blogRefPosts = blogCfg.AssociationReference<Blog, Post>("Posts", true);
            //设置关联引用的本端
            blogRefPosts.HasLeftEnd("End1");
            //设置关联引用的对端
            blogRefPosts.HasRightEnd("End2");
            //设置关联引用属性延迟加载
            blogRefPosts.HasEnableLazyLoading(true);

            //配置实体类型的关联引用属性
            //参数一：关联引用属性的名称 参数二：关联引用是否具有多重性
            //注：此处在配置Post实体与Blog实体关联引用属性Blog
            var postRefBlog = postCfg.AssociationReference<Blog, Post>("Blog", false);
            //设置关联引用的本端（注意此处Post是作为本端的）
            postRefBlog.HasLeftEnd("End2");
            //设置关联引用的对端
            postRefBlog.HasRightEnd("End1");
        }
	}
```


<a name="SzKzD"></a>
### 6. 定义对象集
最终对对象的操作和访问是通过对象上下文提供的对象集，此处我们定义文章和文章评论对象集：
```csharp
    /// <summary>
    /// 自定义对象上下文
    /// </summary>
    public class MyContext : ObjectContext
    {
        /// <summary>
        /// 文章对象集
        /// </summary>
        public ObjectSet<Blog> Blogs { get; set; }

        /// <summary>
        /// 文章评论对象集  
        /// </summary>
        public ObjectSet<Post> Posts { get; set; }
    }
```


<a name="PesP4"></a>
### 7. 对象的创建、读取、更新和删除
<a name="b6igx"></a>
##### 实例化对象上下文
```csharp
var myContext = new MyContext();
```


<a name="5983u"></a>
##### 创建
```csharp
//实例化对象
Blog blog = new Blog()
{
    Url = "https://www.yuque.com/geekfish/obase/getting-started",
    Posts = new List<Post>() {
        new Post (){  Title= "请问Obase怎么安装？", Content = "暂时只提供dll文件"}
    }
};
//将对象附加到对象上下文
myContext.Blogs.Attach(blog);
//将对象保存到数据源
myContext.SaveChanges();
```


<a name="0jarE"></a>
##### 读取
```csharp
using System.Linq;

//从持久化源查询数据
Blog firstBlog = myContext.Blogs.OrderBy(p => p.Url).First();
//访问关联引用属性
List<Post> posts = firstBlog.Posts;
```


<a name="LBsy3"></a>
##### 更新
```csharp
 //修改属性
firstBlog.Url = "http://www.test.com/aa.html";
//将对象保存到数据源
myContext.SaveChanges();
```


<a name="zWNWi"></a>
##### 删除
```csharp
//删除指定对象
myContext.Blogs.Remove(firstBlog);
//根据条件删除指定对象
myContext.Blogs.Delete(p => p.BlogId == 1);
//将对象保存到数据源(只有在保存后，数据才真实删除)
myContext.SaveChanges();
```


