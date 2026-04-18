代码中带有[]的部分为可选部分

- # 数据类型：

	数值型：默认为有符号（正负），可以使用unsigned指定为无符号 int unsigned


- # 整数：

	- ## tinyint：
		- 范围（-128，127）占用1byte
	
	- ## smallint：
		- 范围（-32768，32767）占用2bytes
	
	- ## mediumint：
		- 范围（-8388608，8388607）占用3bytes
	
	- ## int：
		- 范围（-2147483648，2147483647）占用4bytes
	
	- ## bigint：
		- 范围（-2^63,2^63-1）占用8bytes
	
	- ## 浮点数：
		- 创建时可以指定M（精度）D（标度），M是总的位数，D是小数位数float（4，1）总长四位，小数一位，如312.4
	
	- ## float：
		- 范围（-3.408223466 E+38，3.402823466351 E+38）占用4bytes
	
	- ## double：
		- 范围（-1.7976931348623157 E+308，1.7976931348623157 E+308）占用8bytes
	
	- ## decimal：
		- 范围由M和D确定，大小也是


- # 字符串型：

	- ## char类：
		- char：定长字符串，使用时需要指定长度，例如char（10）为创建一个占10字符长度的变量，即使只输入了一个字符，也会占用10个字符的空间，剩下九个使用空格补充，范围：0-255bytes
		
		- varchar：变长字符串，使用时需要指定长度，例如varchar（10）为创建一个最大占10字符长度的变量，占用的空间由其字符数量决定，范围：0-65535bytes
		
	- ## 总结：
		- char的性能好但占用空间固定，varchar性能差但占用空间少，因此在输入如性别这种长度确定的变量时使用char，用户名这种长度不确定的变量时使用varchar


- ## text类：储存文本数据（不常用）

	- ## tinytext：
		- 短文本字符串，范围：0-255bytes
	
	- ## text：
		- 文本字符串，范围：0-65535bytes
	
	- ## mediumtext：
		- 中等长度文本字符串，范围：0-16777215bytes
	
	- ## longtext：
		- 长文本字符串，范围：0-4294967295bytes
	
	- ## blob类：
		- 储存二进制数据（不常用）
	
	- ## tinyblob：
		- 短二进制数据，范围：0-255bytes
	
	- ## blob：
		- 二进制数据，范围：0-65535bytes
	
	- ## mediumblob：
		- 中等长度二进制数据，范围：0-16777215bytes
	
	- ## longblob：
		- 长二进制数据，范围：0-4294967295bytes

- # 日期数据：

	- ## date：
		- 描述日期
		
		- 格式：YYYY-MM-DD
		
		- 范围：1000-01-01到9999-12-31
		
		- 大小：3bytes
	
	- time：格式：HH:MM:SS，描述时间，范围：-838：59：59到838：59：59，大小：3bytes
	
	- year：格式：YYYY，描述年，范围：1901到2155，大小：1bytes（不常用）
	
	- datetime：格式：YYYY-MM-DD HH:MM:SS，范围：1000-01-01 00:00:00到9999-12-31 23:59:59,大小：8bytes
	
	- timestamp：格式：YYYY-MM-DD HH:MM:SS，范围：1970-01-01 00:00:00到2038-1-19 03:14:07，大小:4bytes

- DDL数据库操作：

	- 查询：
	
		- 查询所有数据库：`show databases;`
		
		- 查询当前数据库：`select database();`
	
	- 创建：

		- `create database [if not exists] 数据库名 [default charset 编码类型] [collate 排序规则];`

			- if not exists：意为如果不存在该数据库名，那么创建它，如果存在，不执行操作
	
			- default charset :  指定数据库的编码方式

	- 删除：

		- `drop database [if not exists] 数据库名;`

			- if not exists：意为如果存在该数据库名，那么删除它，如果不存在，不执行操作

	- 使用：
	
		- `use 数据库名；`
	
			- 意为切换到某个要进行操作的数据库，可以使用`select database()`查看当前位于哪个数据库

- DDL表操作-查询与创建：

	- 查询：
	
		- 查询当前数据库所有表：`show tables;`
	
			- 只有处于某个数据库中才可以使用
	
		- 查询表的内容（表结构）：`desc 表名；`
	
			- 会返回表的内容和索引，但是不返回注释
	
		- 查询指定表的建表语句：`show create 表名；`
	
			返回建立该表时输入的代码，包括未指定的默认选项
	
	创建：
	```mysql
	create table 表名(
	
	数据1 数据1类型 [comment '数据1注释'],
	
	数据2 数据2类型 [comment '数据2注释']
	
	)[comment 表注释]
	
	```
	DDL表操作-修改与删除：
	
	添加：
	
	向表格中添加字段：alter table 表名 add 字段名 数据类型（长度）[comment 注释] [约束];
	
	修改：
	
	修改数据类型：alter 表名 modify 字段名 新数据类型（长度）
	
	修改字段名和字段类型：alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];
	
	修改表名：alter table 表名 rename to 新表名；
	
	删除：
	
	删除字段：alter table 表名 drop 字段名；
	
	删除表：drop table [if exists] 表名;
	
	删除并重新创建表：truncate table 表名；