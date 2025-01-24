
## 修改HTTP Proxy

打开Settings，进入Apperance & Behaviour > System Settings > Http Proxy。

勾选Auto-detect proxy settings，填入URL：mirrors.aliyun.com:80

## 修改gradle-wrapper.properties

从项目文件夹进入gradle > wrapper，打开 gradle-wrapper.properties。

访问https://mirrors.aliyun.com/macports/distfiles/gradle/获得所有Gradle版本的下载连接。

修改distributionUrl为：https://mirrors.aliyun.com/macports/distfiles/gradle/gradle-8.7-all.zip。

## 修改settings.gradle.kts

```Kotlin
pluginManagement {
    repositories {
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.aliyun.com/repository/central") }
        maven { url = uri("https://maven.aliyun.com/repository/gradle-plugin") }
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.aliyun.com/repository/central") }
        google()
        mavenCentral()
    }
}

rootProject.name = "My Application"
include(":app")

```