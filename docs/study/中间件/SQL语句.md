# 常见问题 

1.where 和 having的区别

他两的不同还是很好区分的，网上一搜，就能找到，并且能够很好的理解：

Where 是一个约束声明，使用Where约束来自数据库的数据，Where是在结果返回之前起作用的，Where中不能使用聚合函数。
Having是一个过滤声明，是在查询返回结果集以后对查询结果进行的过滤操作，在Having中可以使用聚合函数。
在查询过程中聚合语句(sum,min,max,avg,count)要比having子句优先执行。而where子句在查询过程中执行优先级高于聚合语句。	

| 语句   |   作用时间   | 是否能够使用聚合函数 |
| ------ | :----------: | :------------------: |
| where  | 结果返回之前 |         不能         |
| having | 结果返回之后 |          能          |

参考
https://www.cnblogs.com/Uni-Hoang/p/13235212.html
https://segmentfault.com/a/1190000020742538#:~:text=Where%20%E6%98%AF%E4%B8%80%E4%B8%AA%20%E7%BA%A6%E6%9D%9F%E5%A3%B0%E6%98%8E,%E4%B8%AD%20%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E8%81%9A%E5%90%88%E5%87%BD%E6%95%B0%20%E3%80%82

