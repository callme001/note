## git相关

如何本地创建一个仓库并与github创建的想关联：

```
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:callme001/note.git //直接添加远程仓库地址即可
git push origin master
```