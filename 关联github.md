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

其余操作可根据[博客](https://blog.csdn.net/ClassmateLin/article/details/104576708)进行，就会获得属于自己的网站啦~~

## 本地文件与GitHub项目关联

```
git init
git remote add origin https://github.com/Consck/gitbook.git
```