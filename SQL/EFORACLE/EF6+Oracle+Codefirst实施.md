# VS+EF6+Oracle连接模式

## 实现Service Explorer中的Oracle数据连接

登录ORACLE官网，下载对应Visual Studio版本的ODT Installer。
链接地址为：http://www.oracle.com/technetwork/topics/dotnet/downloads/index.html
>Oracle Data Access Components (ODAC) for Windows Downloads
>ODAC includes Oracle Data Provider for .NET, Oracle Developer Tools for Visual Studio (ODT), Oracle Providers for
>ASP.NET, .NET stored procedure support, as well as additional Oracle data access software for Windows.
>32-bit ODAC with Oracle Developer Tools for Visual Studio Downloads
>32-bit ODAC Xcopy and NuGet Downloads
>64-bit ODAC Downloads - Oracle Universal Installer and Xcopy
>**Oracle Developer Tools for Visual Studio 2017 - MSI Installer**
>**Oracle Developer Tools for Visual Studio 2015 - MSI Installer**
>**Oracle Developer Tools for Visual Studio 2013 - MSI Installer**

在开发PC端安装好相应的ODT后，在VS中的Service Explorer即可创建与ORACLE数据库的数据连接。

![](https://i.imgur.com/aZxdj86.png)  ![](https://i.imgur.com/QPQDlcf.png)

## 实现VS+EF6+Oracle的DBFirst模式

1. 打开VS，新建项目，在NuGet管理解决方案包中搜索Oracle，如下图示：

![](https://i.imgur.com/nnVmSi3.png)

2. 在项目中安装OMDA插件：
>Oracle.ManagedDataAccess V12.2.1100
>Oracle.ManagedDataAccess.EntityFramework V12.2.1100

此外，还需安装EF插件：
> EntityFramework V6.1.3

3. 在项目中添加 **ADO.NET 实体数据模型** 新项，并选择相应数据库中的表格，实现DBFirst模式。

![](https://i.imgur.com/PaErXkU.png)

## 实现VS+EF6+Oracle的CodeFirst模式

1. 打开VS，新建项目，并安装OMDA插件及EF插件。
2. 在config文件中添加连接字符串。
```C#
  <connectionStrings>
    <add name="TestModel" connectionString="DATA SOURCE=192.9.200.194:1521/oraeis;PASSWORD=lis_dev;USER ID=LIS_DEV" providerName="Oracle.ManagedDataAccess.Client" />
  </connectionStrings>
```
3. 新建OracleDbContext.cs。
```C#
 public class OracleDbContext : DbContext
    {
        public OracleDbContext()
            : base("name=TestModel")//使用connectionStrings中定义的连接字符串
        {
        }

        public virtual DbSet<Test> TEST { get; set; }//定义的实体

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.HasDefaultSchema("LIS_DEV");//定义写入的实例
        }
    }
```
4. 实例化OracleDbContext
```C#
 static void Main(string[] args)
        {
            Database.SetInitializer(new DropCreateDatabaseAlways<OracleDbContext>());
            using(var context=new OracleDbContext())
            {
                Test test = new Test() {
                    Id=1,
                    IsStudent=false,
                    name="wanghuan"
                };
                context.TEST.Add(test);
                context.SaveChanges();
            }
        }
```
5. 查看数据库

![](https://i.imgur.com/xUOJHBP.png)    ![](https://i.imgur.com/LN9VHZI.png)