## 1. Guice简介

**1.1 Guice是什么？**

Guice是谷歌推出的一个轻量级依赖注入框架，帮助我们解决Java项目中的依赖注入问题。

**1.2为什么要用？**

如果使用过Spring的话，会了解到依赖注入是个非常方便的功能。不过假如只想在项目中使用依赖注入，那么引入Spring未免大材小用了。

官方解释： Your code will be easier to change, unit test and reuse in other contexts.

Elasticsearch大量使用了Guice

**1.3 怎么用？**

```xml
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>4.1.0</version>
</dependency>
```

官方文档地址： https://github.com/google/guice/wiki/

参考简单入门： https://www.jianshu.com/p/a648322dc680

**1.4** 扩展

Servlet集成
Guice也可以和Servlet项目集成，这样我们就可以不用编写冗长的`web.xml`，以依赖注入的方式使用Servlet和相关组件。

```xml
<dependency>
    <groupId>com.google.inject.extensions</groupId>
    <artifactId>guice-servlet</artifactId>
    <version>4.1.0</version>
</dependency>
```

JSR-330标准
JSR-330是一项Java EE标准，指定了Java的依赖注入标准。Spring、Guice和Weld等很多框架都支持JSR-330。下面这个表格来自于Guice文档，我们可以看到JSR-330和Guice注解基本上可以互换。

Guice官方推荐我们首选JSR-330标准的注解。

| *JSR-330* javax.inject                                       | *Guice* com.google.inject                                    |                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| [@Inject](https://link.jianshu.com?t=http://javax-inject.github.io/javax-inject/api/javax/inject/Inject.html) | [@Inject](https://link.jianshu.com?t=http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Inject.html) | 在某些约束下可互换                     |
| [@Named](https://link.jianshu.com?t=http://javax-inject.github.io/javax-inject/api/javax/inject/Named.html) | [@Named](https://link.jianshu.com?t=http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Named.html) | 可互换                                 |
| [@Qualifier](https://link.jianshu.com?t=http://javax-inject.github.io/javax-inject/api/javax/inject/Qualifier.html) | [@BindingAnnotation](https://link.jianshu.com?t=http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/BindingAnnotation.html) | 可互换                                 |
| [@Scope](https://link.jianshu.com?t=http://javax-inject.github.io/javax-inject/api/javax/inject/Scope.html) | [@ScopeAnnotation](https://link.jianshu.com?t=http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/ScopeAnnotation.html) | 可互换                                 |
| [@Singleton](https://link.jianshu.com?t=http://javax-inject.github.io/javax-inject/api/javax/inject/Singleton.html) | [@Singleton](https://link.jianshu.com?t=http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Singleton.html) | 可互换                                 |
| [Provider](https://link.jianshu.com?t=http://javax-inject.github.io/javax-inject/api/javax/inject/Provider.html) | [Provider](https://link.jianshu.com?t=http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Provider.html) | Guice的Provider继承了JSR-330的Provider |

