django：
![500](assets/DRF/file-20260712180431012.png)

drf：
![500](assets/DRF/file-20260712182517015.png)
![500](assets/DRF/file-20260712183740482.png)![500](assets/DRF/file-20260712193657250.png)
![500](assets/DRF/file-20260712194058377.png)
登录
![500](assets/DRF/file-20260712194619185.png)
全局配置
settings.py:![500](assets/DRF/file-20260712195020698.png)
![500](assets/DRF/file-20260712195240156.png)认证组件不能写在view视图中

权限组件：
![500](assets/DRF/file-20260713104625641.png)
![500](assets/DRF/file-20260713104707042.png)
同样可以在全局注册
![500](assets/DRF/file-20260713105001114.png)
![500](assets/DRF/file-20260713105343240.png)
这里可以写上message以返回无权访问的信息

序列化一个对象
![500](assets/DRF/file-20260714153233863.png)
多个对象
![500](assets/DRF/file-20260714153311564.png)

一键序列化全部对象
![500](assets/DRF/file-20260714153445119.png)

不想让性别返回数字：
![800](assets/DRF/file-20260714154202547.png)

如果有外键，想拿到外键的值：
![800](assets/DRF/file-20260714154528148.png)

不想拿时分秒，只想要年月日：
![500](assets/DRF/file-20260714154707377.png)

数据校验
![500](assets/DRF/file-20260714160828815.png)

内置校验规则
![500](assets/DRF/file-20260714161055336.png)

钩子校验
![500](assets/DRF/file-20260714161524256.png)

