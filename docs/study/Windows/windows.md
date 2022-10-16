# 常用快捷键

参考 

https://jingyan.baidu.com/article/a17d528533aa828099c8f273.html

https://jingyan.baidu.com/article/1974b2898d91b3f4b1f77493.html



1.捕捉快捷键（靠贴窗口）

  下面的快捷键实用来设置你的窗口的

  windows键+上箭头 ：最大化当前窗口到全屏模式。

  windows键+向下箭头 ：恢复窗口的大小，然后最小化窗口。

  windows键+左箭头 ：捕捉当前窗口到屏幕的左半边。

  windows键+右箭头 ：捕捉当前窗口到屏幕的右半边。

   windows键+ 2方向键 ：捕捉当前窗口到屏幕的一个角落（例如Windows +左+上卡窗口，进入左上角）。

2.任务视图：Win + Tab(松开键盘界面不会消失)，可以在窗口和之间切换虚拟桌面 。

  **Ctrl+Alt+Tab快捷键**

  跟Alt+Tab快捷键功能一样，但是松开键盘后，界面不会消失。

  在任务栏间切换应用程序-【Win+T】

3.创建新的虚拟桌面：Win + Ctrl + D

  关闭当前虚拟桌面：Win + Ctrl + F4

4.Win键+A：激活操作中心

5.Win键+E：打开文件管理器

6.Win键+I：打开Windows 10设置

7.Win+L:快速锁屏，

9.Win+Q：快速搜索 或者启动语音小娜。

9.Win键+X：打开高级用户功能

10.Win键+1/2/3：打开任务栏中固定的程序，1代表任务栏中第一个应用图标

11.win＋空格　输入法切换



# 激活

管理员身份打开cmd

```
slmgr /ckms
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms kms.03k.org
slmgr /ato
```



## 蓝屏问题定位

https://bluescreenview.en.softonic.com/





# 开机启动服务

参考

https://blog.51cto.com/u_15338624/3596049#:~:text=1.%E5%BC%80%E5%A7%8B%2D%3E%E8%BF%90%E8%A1%8C%2D,%E8%BF%99%E6%A0%B7%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BA%86%E3%80%82



## 第一种方法：设置启动项

1. 打开windows运行， 输入shell：startup 
    1. 或者找到启动文件夹，我的是C:\Users\Admin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
2. 拷贝需要开机启动的程序的快捷方式到此文件夹即可。

这样就设置好了，下次计算机启动时，程序也会自动启动的。



## 第二种：使用计划任务自启动

...



## 第三种：通过组策略设置脚本随服务器启动

...



## 第四种：添加服务自动运行

...