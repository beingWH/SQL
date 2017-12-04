# SQLServer 角色与权限控制
## 主体与安全对象
1.主体是可以请求系统资源的个体和集合，数据库用户是一种主体，可以按照自己权限的数据库中执行操作和相应的数据。
2.安全对象是指在安全范围内可操作的对象。
>![](https://i.imgur.com/boFEbIx.jpg)
## 元素
* 登录名
只有拥有登录名才能访问实例（sqlserver）

* 角色
角色是一类权限的组合。
>服务器级别角色及其权限
>![](http://images.cnitblog.com/blog/135426/201411/071507313468050.png)
>**注意：** 服务器角色的拥有者只有登录名，服务器角色是固定的，用户无法创建服务器角色。

>数据库级别角色及其权限
>![](http://images.cnitblog.com/blog/135426/201411/071505087377384.png)
>**注意：** 不要将用户创建的数据库角色添加到固定的服务器数据库角色当中去，否则将导致固定的数据库角色的权限升级。

* 用户
用户是数据库级的概念，数据库用户必须绑定具体的登入名，你也可以在新建登入名的时候绑定此登入名拥有的数据库，当你绑定登入名数据库后，数据库默认就创建了此登入名同名的数据库用户，登入名与数据库用户之间就存在关联关系，数据库用户是架构和数据库角色的拥有者，即你可以将某个架构分配给用户那么该用户就拥有了该架构所包含的对象，你也可以将某个数据库角色分配给用户，此用户就拥有该数据库角色的权限。

* 架构
架构是对象的拥有者，架构本身无权限，架构包含数据库对象：如表、视图、存储过程和函数等，平时最常见的默认架构dbo.,如果没指定架构默认创建数据库对象都是以dbo.开头，架构的拥有者是数据库用户、数据库角色、应用程序角色。用户创建的架构和角色只能作用于当前库。

## 实践
### 新建登录名
新建一个登入名person,只给登入名服务器角色分配public权限，不分配数据库。
![](http://images.cnitblog.com/blog/135426/201411/071530522991852.png)
![](http://images.cnitblog.com/blog/135426/201411/071531005035945.png)
接下来用person登入实例，person用户无法访问任何数据库，由于我们未给用户分配任何数据库。

### 给用户分配数据库查看权限
只允许用户查看AdventureWorks2008R2数据库，此时用户可以查询所有对象，但无法修改对象。
![](http://images.cnitblog.com/blog/135426/201411/071614168786171.png)

### 给用户查询某个对象的权限
如果觉得给用户查看权限太大了，将da_datareader数据库角色权限回收，你会发现用户可以访问数据库，但是看不到任何对象。
![](http://images.cnitblog.com/blog/135426/201411/071624410652680.png)
#### 只给用户查看Person.Address表
```
USE AdventureWorks2008R2;
GRANT SELECT ON OBJECT::Person.Address TO person;
--或者使用
USE AdventureWorks2008R2;
GRANT SELECT ON Person.Address TO RosaQdM;
GO
```
![](http://images.cnitblog.com/blog/135426/201411/071636249249512.png)

#### 扩展功能
```
---授予用户person对表Person.Address的修改权限
USE AdventureWorks2008R2;
GRANT UPDATE ON Person.Address TO person;
GO

---授予用户person对表Person.Address的插入权限
USE AdventureWorks2008R2;
GRANT INSERT ON Person.Address TO person;
GO

---授予用户person对表Person.Address的删除权限
USE AdventureWorks2008R2;
GRANT DELETE ON Person.Address TO person;

 --授予用户存储过程dbo.prc_errorlog的执行权限
GRANT EXECUTE ON dbo.prc_errorlog TO person
```
```
标量函数权限：EXECUTE、REFERENCES。

表值函数权限：DELETE、INSERT、REFERENCES、SELECT、UPDATE。

存储过程权限：EXECUTE。

表权限：DELETE、INSERT、REFERENCES、SELECT、UPDATE。

视图权限：DELETE、INSERT、REFERENCES、SELECT、UPDATE。
```
### 授予用户架构的权限
1. 新建数据库角色
![](http://images.cnitblog.com/blog/135426/201411/071750216591898.png)
2. 新增架构
数据库-安全性-架构
![](http://images.cnitblog.com/blog/135426/201411/071754576129392.png)

架构包含数据库对象，我们创建架构persons表
```
---创建架构persons的表
CREATE TABLE Persons.sutdent
(id int not null)
```
用户同时有了Persons.sutdent表的查看权限，因为用户是数据库角色db_person的所有者，而db_person又是架构persons的所有者。
![](http://images.cnitblog.com/blog/135426/201411/071816101285532.png)

创建一些persons架构的视图，存储过程
```
---创建视图
USE AdventureWorks2008R2
GO
CREATE VIEW Persons.vwsutdent
AS
SELECT * FROM Persons.sutdent

GO
USE AdventureWorks2008R2
GO
---创建存储过程
CREATE PROCEDURE Persons.SP_sutdent
(@OPTION NVARCHAR(50))
AS
BEGIN
    SET NOCOUNT ON
    IF @OPTION='Select'
    BEGIN
    SELECT * FROM Persons.sutdent
    END
END
```
![](http://images.cnitblog.com/blog/135426/201411/071829405495244.png)

## 权限的一些方法
### 查询
1. 基础
```
  ---登入名表
  select * from master.sys.syslogins 
  ---登入名与服务器角色关联表
  select * from sys.server_role_members
  ---服务器角色表
  select * from sys.server_principals
  ----查询登入名拥有的服务器角色
  select SrvRole = g.name, MemberName = u.name, MemberSID = u.sid
  from sys.server_role_members m  inner join sys.server_principals g on  g.principal_id = m.role_principal_id 
  inner join sys.server_principals u on u.principal_id = m.member_principal_id

  ---数据库用户表
  select * from sysusers
  ---数据库用户表角色关联表
  select * from sysmembers 
  ---数据库角色表
  select * from sys.database_principals
  ----查询数据库用户拥有的角色
  select ta.name as username,tc.name as databaserole  from sysusers ta inner join sysmembers tb on ta.uid=tb.memberuid
  inner join  sys.database_principals tc on tb.groupuid=tc.principal_id 
```
2. 查询登录名与数据库用户之间的关系
```
--查询当前数据库用户关联的登入名
  use AdventureWorks2008R2
  select ta.name as loginname,tb.name as databaseusername from master.sys.syslogins ta inner join sysusers tb on ta.sid=tb.sid 
  
  /*如果将当前数据库还原到另一台服务器实例上,刚好那台服务器上也存在person登入用户，你会发现二者的sid不一样，
  由于sid不一样,所以登入用户不具有当前数据库的访问权限,我们要想办法将二者关联起来。
  */
  ---关联登入名与数据库用户（将数据库用户的sid刷成登入名的sid）
    use AdventureWorks2008R2
    EXEC sp_change_users_login 'Update_One', 'person', 'person'
    Go
```
3. 查询数据库用户被授予的权限
```
exec sp_helprotect @username = 'person'
```
![](http://images.cnitblog.com/blog/135426/201411/072331059242694.png)
 查询person数据库用户权限会发现，数据库用户拥有的权限都是前面使用GRANT赋予的权限，而后面给用户分配的架构对象不在这个里面显示，上面显示的只是被授予的权限，而架构是数据库用户所拥有的权限。

### 回收
> 如果安全对象是数据库，对应 BACKUP DATABASE、BACKUP LOG、CREATE DATABASE、CREATE DEFAULT、CREATE FUNCTION、CREATE PROCEDURE、CREATE RULE、CREATE TABLE 和 CREATE VIEW。
> 如果安全对象是标量函数，对应 EXECUTE 和 REFERENCES。
> 如果安全对象是表值函数，对应 DELETE、INSERT、REFERENCES、SELECT 和 UPDATE。
> 如果安全对象是存储过程，表示 EXECUTE。
> 如果安全对象是表，对应 DELETE、INSERT、REFERENCES、SELECT 和 UPDATE。
> 如果安全对象是视图， 对应 DELETE、INSERT、REFERENCES、SELECT 和 UPDATE。

- 回收dbo.prc_errorlog存储过程的执行权限
```
USE AdventureWorks2008R2;
REVOKE EXECUTE ON dbo.prc_errorlog    FROM person;
```
![](http://images.cnitblog.com/blog/135426/201411/072343037215603.png)
- 回收Person.Address表的查询，修改，删除权限
```
--回收修改
USE AdventureWorks2008R2;
REVOKE update ON   Person.Address FROM person;

USE AdventureWorks2008R2;
REVOKE alter ON   Person.Address FROM person;

--回收删除
USE AdventureWorks2008R2;
REVOKE delete ON   Person.Address FROM person;

--回收查询
USE AdventureWorks2008R2;
REVOKE select ON   Person.Address FROM person;
```
![](http://images.cnitblog.com/blog/135426/201411/072347200819482.png)
最后剩下owner为‘.’的是数据库级的权限
```
USE AdventureWorks2008R2;
REVOKE CREATE TABLE FROM person;
GO

CONNECT权限是用户访问数据库的权限，将此权限回收后用户将无法访问数据库
--USE AdventureWorks2008R2;
--REVOKE CONNECT FROM person;
--GO
```
再执行exec sp_helprotect @username = 'person'，就剩下action=connect的数据库访问权限
![](http://images.cnitblog.com/blog/135426/201411/080059260031923.png)
将权限回收后，数据库用户还剩下架构Persons的权限，如果还需要将该权限回收，只需要用户取消关联对应的db_person数据库角色权限。

## 补充
针对生产数据库服务器创建一个应用程序访问的用户最常见的是授予用户某个数据库：“查询”、“删除”、“修改”、“插入”、“执行”的权限，用SQL语句实现如下（用户：person,数据库：news）：

```
USE [master]
GO
---创建登入名
CREATE LOGIN [person] WITH PASSWORD=N'person', DEFAULT_DATABASE=[news], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO
USE [news]
GO
---在指定的数据库下创建和登入名相关联的数据库用户
CREATE USER [person] FOR LOGIN [person]
GO
USE [news]
GO
---在指定的数据库下授予用户SELECT,DELETE,UPDATE,INSERT,EXECUTE权限。
GRANT SELECT,DELETE,UPDATE,INSERT,EXECUTE TO person;
```
注意：创建登入名在master数据库下，创建数据库用户和授予数据库权限都是在具体的数据库下操作。







