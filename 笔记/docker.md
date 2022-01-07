mysql操作命令：
	https://www.cnblogs.com/zhangzhu/p/3172486.html
	
查询数据库全部用户信息：
     select user,host from mysql.user;

修改数据表某一列字段类型：
	ALTER TABLE `push_target` MODIFY COLUMN id BIGINT(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增长主键';
	
数据记录插入：
insert into device_tool_display (type,description,picture,sub_page,display_name,priority) values ('9','适配本地U口打印机、网口打印机，支持打印扫码点单、外卖等小票','https://images.wosaimg.com/c9/90656736d92ab575ce77549b88fab5f53bf3f6.png','https://iot-audio-app.shouqianba.com/print/bind/assistant','打印助手','1');

清空表数据且自增长ID从1开始：
truncate table cm_push_target;

创建数据表：
CREATE TABLE `cm_push_target` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增长主键',
  `store_sn` varchar(64) NOT NULL DEFAULT '' COMMENT '门店Sn',
  `client_id` varchar(64) NOT NULL DEFAULT '' COMMENT '设备ID',
  `disable` tinyint(1) DEFAULT '1' COMMENT '是否有效',
  `ctime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `mtime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `UNIQ_STORE_CLIENT_ID` (`store_sn`,`client_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='超盟设备信息表';


更新一条数据库记录：
update printer_detail_info set paper_spec = 2 where paper_spec = 80; 
update `device_store_info` set status = 0 where device_sn = "CH1I0A0713294219L";



##启动MySQL服务
sudo /usr/local/MySQL/support-files/mysql.server start
##停止MySQL服务
sudo /usr/local/mysql/support-files/mysql.server stop
##重启MySQL服务
sudo /usr/local/mysql/support-files/mysql.server restart



mysql忘记密码：

1.苹果->系统偏好设置->最下边点mysql 在弹出页面中 关闭MySQL服务（点击stopMySQL server）。

2.进入终端输入：cd /usr/local/mysql/bin/

回车后 登录管理员权限 sudo su

回车后输入以下命令来禁止mysql验证功能 ./mysqld_safe --skip-grant-tables & （别漏掉最前的" . "）

回车后mysql会自动重启（偏好设置中MySQL的状态会变成running）

3.输入命令 ./mysql

回车后，输入命令 FLUSH PRIVILEGES; 

回车后，输入命令 SET PASSWORD FOR 'root'@'localhost' = PASSWORD('你的新密码');

4.可以用新密码登录MySQL数据库。


查看数据库进程：
ps -ef|grep mysqld

杀死进程：
kill -9 进程号


查看正在运行的docker容器：
docker ps | grep '服务名'


查看实时docker日志：
docker logs -f --tail=10 888f24e2c52c

10：最后10行；888f24e2c52c：docker文件名

重启docker服务：
sudo docker restart mqtthoo

回退命令：
$ git reset --hard HEAD^ 回退到上个版本
$ git reset --hard HEAD~3 回退到前3次提交之前，以此类推，回退到n次提交之前
$ git reset --hard commit_id 退到/进到 指定commit的sha码
强推到远程：
$ git push origin HEAD --force


项目迁移
重复操作会自动覆盖，此迁移方式会将所有的分支和tag一并移动
git clone --mirror <URL to my OLD repo location>
cd <New directory where your OLD repo was cloned>
git remote set-url origin <URL to my NEW repo location>
git push -f origin