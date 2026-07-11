## 基础

git config --global user.name []         设置用户签名

git config --global user.email []         设置用户签名
![500](assets/git/file-20260711131910259.png)

**git init              初始化本地库**
![500](assets/git/file-20260711131910258.png)

git status        查看状态
![500](assets/git/file-20260711131910256%201.png)
![500](assets/git/file-20260711131910257%201.png)

git add         添加暂存区
![500](assets/git/file-20260711131910258%201.png)
![500](assets/git/file-20260711131910251%201.png)

git rm --cached       删除暂存区的文件（工作区没被删除）
![500](assets/git/file-20260711131910251.png)

git commit -m "日志信息" 文件名     提交本地库
![500](assets/git/file-20260711131910257.png)
+ 其中高亮显示的是七位的版本号
![500](assets/git/file-20260711131910250.png)

git reflog    或者    git log        查看版本信息
![500](assets/git/file-20260711131910256.png)

修改文件并提交：
![500](assets/git/file-20260711131910245.png)

git reset --hard 版本号          穿梭版本


## 分支
gti bransh -v         查看分支
git branch 分支名        创建分支
![500](assets/git/file-20260711131910240.png)

git chekout        切换分支
![500](assets/git/file-20260711131910235.png)
![500](assets/git/file-20260711131910230.png)

gti merge 要合并的分支
![500](assets/git/file-20260711131910215.png)



## 团队协作
#### 团队内协作：
![500](assets/git/file-20260711131910220.png)
##### 添加成员到自己的团队
![500](assets/git/file-20260711131910174.png)

#### 跨团队协作：
![500](assets/git/file-20260711131910225.png)
##### fork到自己仓库修改
![500](assets/git/file-20260711131910160.png)

## GitHub操作
创建仓库
https://github.com/gbx200/gitdemo.git
![500](assets/git/file-20260711131910204.png)

git remote add 别名 http          创建别名
git remote -v                            查看别名
![500](assets/git/file-20260711131910196.png)

git push 别名 分支名                推送
![500](assets/git/file-20260711131910210.png)
![500](assets/git/file-20260711131910183.png)

git pull 别名 分支名                 拉取库到本地
![500](assets/git/file-20260711131910190.png)

git clone https：//······           克隆仓库（不需要登录）

## IDEA集成git
#### 配置忽略文件
	1.创建忽略规则文件git.ignore

#### 创建git仓库
![500](assets/git/file-20260711131910159.png)
##### 添加暂存区
![500](assets/git/file-20260711131910158.png)

#### 提交本地库
![500](assets/git/file-20260711131910157%201.png)

#### 切换版本
![500](assets/git/file-20260711131910157.png)

#### 创建分支
![500](assets/git/file-20260711131910156.png)

#### 合并分支
1. 在hotfix修改
2. 提交修改到本地库
3. 切换回主干分支
4. 
 ![500](assets/git/file-20260711131910155.png)

#### 登录GitHub
![500](assets/git/file-20260711131910151.png)

#### 分享到github
![500](assets/git/file-20260711131910150.png)
选择share

