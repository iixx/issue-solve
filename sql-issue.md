mysql的 `str_to_date` 函数转换空字符串会变成 `0000-00-00` 这种非法日期,
数据库连接url可添加参数 `zeroDateTimeBehavior=convertToNull` 修正此问题.

---

Oracle迁移到mysql的问题:

当列是`int`类型时插入的值不允许为'', 需改为NULL. 这种问题一般mysql 5.x上出现.  
但如果这样修改, 其他sql语句也会暗藏bug, 很可能改不全.  
官方解释说: 得知新版本mysql对空值插入有"bug", 要在安装mysql的时候去除默认勾选的enable strict SQL mode  
或修改mysql的配置文件,将
	`sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"`
改为
	`sql-mode="NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"`

---

附新出现的问题:

由于需要把Oracle的insert blob改为mysql支持的方式,经测试发现jdbc驱动需要从`mysql-connector-java-3.1.12-bin.jar`更新为`mysql-connector-java-5.1`以上,
更新后insert blob成功,但会再次出现 `int类型时不允许''` 的问题.  
使用 `SELECT @@GLOBAL.sql_mode;` 和 `SELECT @@SESSION.sql_mode;` 确认全局设置没有问题, session里会出现严格模式STRICT_TRANS_TABLES.  
原因如下:
>	The driver needs the STRICT_TRANS_TABLES mode enabled to enforce JDBC compliance on truncation checks.
	If you can't use STRICT_TRANS_TABLES as part of your sql_mode, then you'll have to disable truncation checks by adding "jdbcCompliantTruncation=false" as a URL configuration parameter.

因此url参数需要增加`jdbcCompliantTruncation=false`.

参考源:http://stackoverflow.com/questions/590937/mysql-coldfusion-8-sql-mode