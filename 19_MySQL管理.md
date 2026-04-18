系统数据库：
	MySQL数据库安装完成后自带四个数据库：
		mysql：存储mysql服务器正常运行所需各种信息（时区，主从，用户，权限等）
		information_schema：提供了访问数据库元数据的各种视图和表，包含数据库，表，字段类型及访问权限等
		performance_schema：为MySQL服务器运行时状态提供了一个底层监控功能，主要用于搜集数据库服务器性能参数
		sys：包含了一系列方便DBA和开发人员利用performance_schema性能数据库进行性能调优和诊断的视图
常用工具：
	mysql：
```
		语法：mysql [options] [databse]
		选项：
			-u 
```