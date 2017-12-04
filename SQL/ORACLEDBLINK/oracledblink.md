# Oracle DBLINK 创建

## 创建位置
如果是从A端对B端进行访问，应在A端创建相应的DBLINK。

## 实施方法
1. `tnsnames.ora`文件
	在A端oracle安装路径`D:\oraclexe\app\oracle\product\11.2.0\server\network\ADMIN`中找到文件`tnsnames.ora`，并增加对B端的HOST及PORT的描述，具体如下：
	```
	XE_MID =
	  (DESCRIPTION =
	    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.9.250.54)(PORT = 1521))
	    (CONNECT_DATA =
	      (SERVER = DEDICATED)
	      (SERVICE_NAME = XE)
	    )
	  )
	```
	保存文件后，在CMD中可以使用`tnsping XE_MID`来查看是否能够ping通B端。
2. 公共数据库链接
	在A端创建`Public DBLINK`，具体如下：

	名称|值|备注
	-----|-----|-----
	OWNER|PUBLIC|标识为公共的链接
	DB_LINK|XE_MID.WORLD|用于链接B端的链接名
	USERNAME|ATIS_DEV|B端具备相应权限的SCHEMA的USERNAME
	PASSWORD|atis_dev|B端具备相应权限的SCHEMA的PASSWORD
	HOST|XE_MID|A端tnsnames文件中使用的配置参数
	
	创建后可以查看其SQL，如下：
	```
	CREATE PUBLIC DATABASE LINK "XE_MID.WORLD"
	   CONNECT TO "ATIS_DEV" IDENTIFIED BY VALUES '052CFB61DDD7015E259F8CC72C3DCE73A712FD46EEB169323F'
	   USING 'xe_mid';

	```
3. 数据查询
	在A端SCHEMA下使用如下方法进行查询：
	```
	SELECT * FROM ATIS_DEV."OTimerCfg"@"XE_MID.WORLD";
	# 注意双引号
	```

	在A端SCHEMA下使用如下方法进行更新：
	```
	UPDATE "ATIS_DEV"."OTimerCfg" @"XE_MID.WORLD" SET "TimerName" = 'SDDSADSADSA' WHERE "Id"='25'
	```

	在A端SCHEMA下使用如下方法进行插入：
	```
	INSERT INTO "ATIS_DEV"."OTimerCfg" @"XE_MID.WORLD" ("Id", "TimerName", "TimerMark", "TimerSwitch") VALUES ('13', 'sfds', 'sfdfsdsd', '1')
	```

	在A端SCHEMA下发起Merge操作：
	```
	MERGE INTO "ATIS_DEV"."OTimerCfg"@"XE_MID.WORLD"
	USING "ROLLER_WORK"."TimerCfg"
	on("OTimerCfg"."Id"="TimerCfg"."Id")
	when matched then
	update set "OTimerCfg"."TimerName"="TimerCfg"."TimerName","OTimerCfg"."TimerMark"="TimerCfg"."TimerMark","OTimerCfg"."TimerSwitch"="TimerCfg"."TimerSwitch"
	when not matched then
	insert("OTimerCfg"."Id","OTimerCfg"."TimerName","OTimerCfg"."TimerMark","OTimerCfg"."TimerSwitch")Values("TimerCfg"."Id","TimerCfg"."TimerName","TimerCfg"."TimerMark","TimerCfg"."TimerSwitch")
	```