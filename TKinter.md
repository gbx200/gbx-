### 主窗口：
```python
import tkinter as tk

# 创建主窗口
root = tk.TK()
root.title("系统时钟")  # 名称
root.geometry("400x100+200+200")  # 窗口大小

# 进入事件主循环
root.mainloop()
```
### 标签（Label）
```python
# 创建标签
time_label = tk.Label(root, text="08:41:04", font=("Arial", 60),fg="red")
# 将标签添加到主窗口
time_label.pack()
```
### 按钮（Button）
```python
def button_click():
	print("按钮被点击了")
	label.config(text="按钮被点击了")
	
button = tk.Button(root, text="点击我", command=button_click)

button.pack()
```
### 单行文本框（输入框）（Entry）
```python
entry = tk.Entry(root, width=30)
entry.pack

# 获取输入的值
def button_click2():  
    value = entry.get()  
    time_label.config(text=value)  
button1 = tk.Button(root, text="获取输入", command=button_click2)  
button1.pack()
```
### 多行文本框（Text）
```python
text = tk.Text()
```