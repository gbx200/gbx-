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
text = tk.Text(root, height=5, width=30)  
text.pack()

def get_text_value():  
    value = text.get("1.0", tk.END)  # <- 从第一行第零列到末尾
    print(f'文本框的值是：{value}')  
button2 = tk.Button(root, text="获取输入", command=get_text_value)  
button2.pack()
```
### 单选按钮（Radiobutton）
```python
var = tk.StringVar(value="A")  # 默认选择A
radio1 = tk.Radiobutton(root, text="选项A", variable=var, value="A")
radio2 = tk.Radiobutton(root, text="选项B", variable=var, value="B")
radio1.pack()
radio2.pack()
# 获取选择的值
def get_radio_value():
    print(f"选择的值是: {var.get()}")
button = tk.Button(root, text="获取选择", command=get_radio_value)
button.pack()
```
### 复选框（Checkbutton）
```python
import tkinter as tk  
  
root = tk.Tk()  
root.title("复选框控件的使用")  
root.geometry("400x300+200+200")  
  
var1 = tk.IntVar()  
var2 = tk.IntVar()  
var3 = tk.IntVar()  
var4 = tk.IntVar()  
  
check1 = tk.Checkbutton(root, text="篮球", variable=var1)  
check1.pack()  
check2 = tk.Checkbutton(root, text="足球", variable=var2)  
check2.pack()  
check3 = tk.Checkbutton(root, text="羽毛球", variable=var3)  
check3.pack()  
check4 = tk.Checkbutton(root, text="排球", variable=var4)  
check4.pack()  
  
def get_hobby():  
    hobby = ""  
    if var1.get() == 1:  
       hobby += "篮球 "    if var2.get() == 1:  
       hobby += "足球 "    if var3.get() == 1:  
       hobby += "羽毛球 "    if var4.get() == 1:  
       hobby += "排球 "    label.config(text="您选择的兴趣爱好是：" + hobby)  
  
button = tk.Button(root, text="获取选中的选项", command=get_hobby)  
button.pack()  
label = tk.Label(root, text="您选择的兴趣爱好是：")  
label.pack()  
  
root.mainloop()
```
