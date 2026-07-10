git config --global user.name []         设置用户签名

git config --global user.email []         设置用户签名
![500](assets/git/file-20260710192526507.png)

**git init              初始化本地库**
![500](assets/git/file-20260710192857767.png)

git status        查看状态
![500](assets/git/file-20260710193057023.png)
![500](assets/git/file-20260710193332287.png)

git add         添加暂存区
![500](assets/git/file-20260710193522248.png)
![500](assets/git/file-20260710193537110.png)

git rm --cached       删除暂存区的文件（工作区没被删除）
![500](assets/git/file-20260710193706536.png)

git commit -m "日志信息" 文件名     提交本地库
![500](assets/git/file-20260710193858591.png)
+ 其中高亮显示的是七位的版本号
![500](assets/git/file-20260710193955951.png)

git reflog    或者    git log        查看版本信息
![500](assets/git/file-20260710194150561.png)

修改文件并提交：
![500](assets/git/file-20260710194500848.png)

git reset --hard 版本号          穿梭版本
