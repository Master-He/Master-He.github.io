# 使用 Kotlin DSL 的 Gradle 中央仓库镜像源配置

参考https://ld246.com/article/1608277432537



## 使用 Kotlin DSL 配置普通依赖镜像源

修改 `build.gradle.kts` 文件加入以下内容：

```kts
repositories {
    // 依赖使用阿里云 maven 源
    maven {
        setUrl("https://maven.aliyun.com/repository/public/")
    }
    maven {
        setUrl("https://maven.aliyun.com/repository/spring/")
    }
    mavenLocal()
    mavenCentral()
}

```

## 使用 Kotlin DSL 配置插件依赖镜像源

修改 `settings.gradle.kts` 文件加入以下内容：

```kts
pluginManagement {
    repositories {
        // 插件使用阿里云 maven 源
        maven {
         setUrl("https://maven.aliyun.com/repository/gradle-plugin")
        }
    }
}
```



# Groovy DSL

## 使用 Groovy DSL 配置普通依赖镜像源

修改 `build.gradle` 文件：

```groovy
allProjects {
  repositories {
    maven {
      url 'https://maven.aliyun.com/repository/public/'
    }
    maven {
        url 'https://maven.aliyun.com/repository/spring/'
    }
    mavenLocal()
    mavenCentral()
  }
}

```

## 使用 Groovy DSL 配置插件依赖镜像源

修改 `settings.gradle` 文件加入以下内容：

```groovy
pluginManagement {
    repositories {
        // 插件使用阿里云 maven 源
        maven {
            url 'https://maven.aliyun.com/repository/gradle-plugin'
        }
    }
}
```