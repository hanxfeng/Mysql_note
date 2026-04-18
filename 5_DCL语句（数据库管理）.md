用户管理：

查询用户：

use mysql;

select * from user;

因为mysql的数据都储存在mysql数据库的user表中，所以使用use进入后使用select即可查询

创建用户：

create user '用户名'@'主机名' identified by '密码';

主机名有两种，localhost只能在本机登录，%能在任意机器登录

密码必须要在单引号中，不过输入时不用加单引号

修改用户密码：

alter user  '用户名'@'主机名' inentified with mysql_native_password by '新密码';

masql中使用主机名+用户名来确定用户

删除用户：

drop user  '用户名'@'主机名' ;

权限控制：

常用权限及其说明：

查询权限：

show grants for  '用户名'@'主机名';

授予权限：

grant 权限列表 on 数据库名.表名 to  '用户名'@'主机名'；

如果要授予所有数据库和表权限，那么可以使用 *.* 代替

撤销权限：

revoke 权限列表 on 数据库名.表名 from  '用户名'@'主机名'；