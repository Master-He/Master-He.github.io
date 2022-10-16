

# Tomcat问题

## Tomcat乱码问题

参考 https://blog.csdn.net/B2345012/article/details/88365722

1. 找到**Tomcat**目录下**conf**文件夹中的**logging.properties**文件
2. 打开**logging.properties**文件，找到文件中的**java.util.logging.ConsoleHandler.encoding = UTF-8**
3. 将其中的**UTF-8**改为**GBK**，保存后重启Tomcat服务，启动后就会看到刚才的乱码已经转换过来了。





项目创建

![image-20221016170323024](Tomcat.assets/image-20221016170323024.png)

项目结构参考：

https://github.com/Master-He/java-demo  下的servlet-demo/src/main/java/org/github/servletdemo/HelloServlet.java

![image-20221016172921195](Tomcat.assets/image-20221016172921195.png)





tomcat配置

添加tomcat local

![image-20221016173304211](Tomcat.assets/image-20221016173304211.png)

![image-20221016173439066](Tomcat.assets/image-20221016173439066.png)



![image-20221016173502401](Tomcat.assets/image-20221016173502401.png)

![image-20221016173530083](Tomcat.assets/image-20221016173530083.png)

![image-20221016173752263](Tomcat.assets/image-20221016173752263.png)