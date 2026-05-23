## 后端架构搭建
1. 创建django项目
2. 在settings.py中删除如下内容：![400](assets/django+vue3前后端分离初始化项目/file-20260523151309967.png)
	此admin是django自带的管理系统，防止请求冲突
3. 依次点击**工具，运行**

 ![400](assets/django+vue3前后端分离初始化项目/file-20260523151517561.png)
4. 在底部输入以下内容来创建app，user可以更改为app名
	![400](assets/django+vue3前后端分离初始化项目/file-20260523151801210.png)
5. 在settings.py中添加以下内容![400](assets/django+vue3前后端分离初始化项目/file-20260523152042082.png)
6. 管理url
+ 
+ 在你的app中创建url.py并写入以下代码