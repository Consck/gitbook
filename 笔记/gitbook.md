# gitbook安装过程(mac)

## 安装node

[下载](https://nodejs.org/en/blog/release/v10.22.0/)10.22.0版本pkg文件。否则容易出现版本问题引起的[错误](https://www.jianshu.com/p/c75381320b8a)

安装完成后使用命令验证

```
$node -v
v10.22.0
$npm -v
6.14.6
```

## 安装gitbook

`sudo npm install gitbook-cli -g`


通过命令`gitbook -V`验证

```
CLI version: 2.3.0
GitBook version: 3.2.2
```

## gitbook使用

1. 创建文件夹，作为gitbook编写大本营
1. 编写test.md文件,并将目录加入SUMMARY.md文件中
1. 运行命令gitbook init
1. 运行命令gitbook build
1. 运行命令gitbook serve
1. 通过http://localhost:4000查看

