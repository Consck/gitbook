# 命令框
```linux
# csdn @ edu in ~ [11:32:59] 
$ 
```
1. csdn: 表示当前的登陆用户,这里是使用csdn账户登陆.
2. @: 分隔符.
3. edu: 主机短名称
4. ~: 当前所在目录
5. $: 命令提示符.如果是root用户提示符是#;Linux用这个符号标识登陆用户的权限.

# liunx命令

* 创建文件夹：mkdir fileName
* 创建空白文件：touch fileName
* 删除空白文件：rm fileName [rm --help]
* 单个命令删除多个目录：rm -rf dir1 dir2 dir3
  * 递归删除目录，使用选项-r或-R
* 删除空文件夹：rmdir fileName
* 复制fileName1文件夹并命名为fileName2：cp -r fileName1 fileName2
  * -r 表示操作目录,如果是文件则不需要加-r.
  * Linux对大小写是严格区分的. 
* 移动文件：mv fileName1 fileName2/
* 文件重命名：mv oldName newName
* 查看文件内容：cat [-n] fileName 
  * -n：加行号
* 查看文件前十条内容：head [-n num] fileName 
  * -n num：查看文件前num行内容
  * tail默认查看文件后十行内容
* 编写文件内容：vim
* 查找文件：find /etc/ -name passwd
    ```linux
    #find 目录 -user 指定用户名
    find /etc  -user root
    #find 目录 -size 文件大小
    find /etc -size 1M
    ```
* 创建软链接快捷方式：ln -s originfile /home/csdn/myusr
  * eg展示方式[ls -l]：myusr -> /usr
* 压缩一个文件：gzip fileName
  * 此种方式只能将文件压缩成*.gz格式
* 解压文件：gunzip fileName
* 安装tree软件：sudo yum install tree
  * sudo 可以理解为暂时拥有管理员权限。权限会在进阶课程详细说明。
  * yum 是centos下的安装工具
  * 在屏幕输出家目录的树形图:tree /home

* 去掉表头：sed 1d test.csv > te.csv
* CSV文件去掉空行：awk NF te.csv > t.csv
* 排序去重并输出CSV: sort t.csv |uniq > re.csv
* 切割CSV文件：split -a 2 -l 1 re.csv wwwwwww   
	* [-a 2]: 文件后缀长度为2，以 aa、ab、ac 这样的双字母格式作为后缀
	* [-l 1]： 每隔一行切割为一个文件
* 对切割后的CSV文件添加首行表头：
* 对$$切割后的文件进行重命名：for i in *; do mv $i $i".csv"; done

# shell脚本编程

> #! 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序
echo 命令用于向窗口输出文本



# awk

# perl 

# Mac下查看内存
* top -l 1 | head -n 10 | grep PhysMem

Linux下使用free命令可以查看内存