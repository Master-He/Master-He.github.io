可选框架

```
pytest(兼容nose和unittest, 基本都用这个，但需要安装)
nose(需要安装)
unittest(不需要安装， 所以用的人也多)
```



# unittest介绍
unittest是python内置的用于测试代码的模块，无需安装， 使用简单方便。类似于Java的JUnit，是JUnit的Python版本。

unittest 整体来讲分为如下几个大的模块：

## 1. test fixture（测试脚手架）
test fixture 表示为了开展一项或多项测试所需要进行的准备工作，以及所有相关的清理操作。举个例子，比如执行前连接数据库、打开浏览器等，执行完成后需要还原数据库、关闭浏览器等操作。这时候就可以启用testfixture。

setUp()：准备环境，执行每个测试用例的前置条件。对应于JUnit的@Before；
tearDown()：环境还原，执行每个测试用例的后置条件。对应于JUnit的@After；
setUpClass()：必须使用@classmethod装饰器，所有case执行的前置条件，只运行一次。对应于JUnit的@BeforeClass；
tearDownClass()：必须使用@classmethod装饰器，所有case运行完后只运行一次。对应于JUnit的@AfterClass；

## 2. test case（测试用例）

一个测试用例是一个独立的测试单元。它检查输入特定的数据时的响应。 unittest 提供一个基类： TestCase，用于新建测试用例。源码：unittest/case.py

一个类class继承 unittest.TestCase，就是一个测试用例。一个TestCase的实例就是一个测试用例，就是一个完整的测试流程。包括测试前环境准备setUp()|setUpClass()、执行代码run()、测试环境后的还原tearDown()|tearDownClass()。

继承自unittest.TestCase的类中，测试方法的名称要以test开头。默认只会执行以test开头定义的方法（测试用例）。这一点，可以从它的源码中看出来（unittest/loader.py）

TestCase中提供了很多检查条件和报告错误的方法，都是以assert开头。

当由于某些原因我们不想去执行或者需要达到某些条件时候采取执行一些case，TestCase中提供了跳过case 的一些装饰器:

skip, skipIf  skipUnless



## 3. test suite（测试套件）

test suite 是一系列的测试用例，或测试套件，或两者皆有。它用于归档需要一起执行的测试。

对于用例的执行顺序，默认按照test的 A-Z、a-z的方法执行。若要按自己编写的用例的先后关系执行，需要用到testSuite。把多个测试用例集合起来，一起执行，就是testSuite。testsuite还可以包含testsuite。




## 4. test loader（测试加载器）

test loader用来加载test case到test suite中。

loadTestsFrom*()方法从各个地方寻找test case，创建实例，然后addTestSuite，再返回一个TestSuite实例。

该类提供以下方法：

- unittest.TestLoader().loadTestsFromTestCase(testCaseClass)
- unittest.TestLoader().loadTestsFromModule(module)
- unittest.TestLoader().loadTestsFromName(name,module=None)
- unittest.TestLoader().loadTestsFromNames(names,module=None)
- unittest.TestLoader().getTestCaseNames(testCaseclass)
- unittest.TestLoader().discover()

test loader默认加载以test开头的文件。可以从源码中看出来



## 5. test runner（测试运行器）

test runner 是一个用于执行和输出测试结果的组件。这个运行器可能使用图形接口、文本接口，或返回一个特定的值表示运行测试的结果。

unittest默认提供了 TextTestRunner（源码：unittest/runner.py），也可以自己实现TestRunner。比较好用的第三方的TestRunner为：HTMLTestRunner，可以自动生成HTML页面的测试报告。

TestRunner可以接收test case，也可以接收test suite



## 6. test result（测试结果）

test result 会保存所有用例的运行结果。



## 总结6大组件的关系如下

![img](Python单元测试.assets/0f14395232babcebbf10fd9c40fff6fe.png)





unittest默认提供了 TextTestRunner（源码：unittest/runner.py），也可以自己实现TestRunner。比较好用的第三方的TestRunner为：[HTMLTestRunner](http://tungwaiyip.info/software/HTMLTestRunner.html)，可以自动生成HTML页面的测试报告。

```python
t_suite1 = unittest.TestSuite()
t_suite1.addTests([SomeoneClass('someone_method')])

t_suite2 = tunittest.defaultTestLoader.discover(testcase_path, pattern=model_pattern)
```



github比较好的项目： https://github.com/search?q=python+unittest+framework





python 测试框架 内网总结

http://tungwaiyip.info/software/HTMLTestRunner.html







