## 以gitbook方式显示GitHub中的md文件

GitHub为每一个用户都分配了域名{用户名}.github.io

新建repositories 名为：Consck.github.io

新建一个文件夹，把项目拉取下来并执行：

```
git clone https://github.com/Consck/Consck.github.io.git
printf "<h1>Consck's HomePage</h1>It works.\n" > index.html
git add index.html
git commit -m "Homepage test version."
git push origin master
```

其余操作可根据[博客](https://blog.csdn.net/ClassmateLin/article/details/104576708)进行，不需要全部按照描述操作。将本地md文件夹与GitHub项目关联后，新建分支名字为gh-pages(我也不知道为啥非得叫这个名字，不然最后打开的网址没有左侧目录栏)。之后需要将文件夹中的文件上传至master和gh-pages分支。

## 本地文件与GitHub项目关联
[参考链接](https://www.jianshu.com/p/37d4276c857a)
```
git init
git remote add origin https://github.com/Consck/gitbook.git
```

## 以脚本的方式执行上传动作

gitbook.sh文件内容：

```
cd /Users/macuser/Desktop/gitbook-study
gitbook build
git add .
git commit -m 'update gitbook'
git push -u origin master
git checkout gh-pages
rm -rf *
git checkout master -- _book
mv _book/* ./
rm -rf _book
rm -rf publish.sh
git add .
git commit -m 'publish gh-pages'
git push origin gh-pages
git checkout master
```

脚本文件为gitbook.sh文件，在cmd中执行：`./gitbook.sh`即可

**注意**：使用时不能将脚本放在仓库里，不然在切换分支时容易出错，最好与仓库文件夹同级。