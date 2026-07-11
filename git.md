## 基础

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


## 分支
gti bransh -v         查看分支
git branch 分支名        创建分支
![500](assets/git/file-20260710200335280.png)

git chekout        切换分支
![500](assets/git/file-20260710200449310.png)
![500](assets/git/file-20260710200845928.png)

gti merge 要合并的分支
![500](assets/git/file-20260710201025855.png)



## 团队协作
#### 团队内协作：
![500](assets/git/file-20260710202253504.png)
##### 添加成员到自己的团队
![500](assets/git/file-20260711101516697.png)

#### 跨团队协作：
![500](assets/git/file-20260710202630632.png)
##### fork到自己仓库修改
![500](assets/git/file-20260711102356712.png)

## GitHub操作
创建仓库
https://github.com/gbx200/gitdemo.git
![500](assets/git/file-20260710203206577.png)

git remote add 别名 http          创建别名
git remote -v                            查看别名
![500](assets/git/file-20260710203604431.png)

git push 别名 分支名                推送
![500](assets/git/file-20260710204010694.png)
![500](assets/git/file-20260710204221415.png)

git pull 别名 分支名                 拉取库到本地
![500](assets/git/file-20260710204538641.png)

git clone https：//······           克隆仓库（不需要登录）

