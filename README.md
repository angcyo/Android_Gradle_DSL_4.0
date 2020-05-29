# Android_Gradle_DSL_4.0

`gradle 4.0`源码分析, 源码环境:

`com.android.tools.build:gradle:4.0.0`

`gradle-6.1.1-all`


# 配置工程

## gradle 源码

复制`gradle 6.1.1` 源码到工程的`gradle-6.1.1`目录下.

### Gradle 源码所在路径

**MAC**

> /Users/用户名/.gradle/wrapper/dists

**Win**

> C:/Users/用户名/.gradle/wrapper/dists

## gradle 插件

也就是 `com.android.tools.build:gradle:4.0.0` 这个jar包.

复制`com.android.tools.build:gradle:4.0.0`jar包到项目工程的`libs`文件夹, 这个普通的安卓工程引入jar包方式一致.

通常要看源码的话, 还需要`xxx-sources.jar`一起复制.

还需要的jar包(相同文件夹路径下的):
`com.android.tools.build/gradle-api`

## jar包路径

**MAC**

```java
/Users/用户/.gradle/caches/modules-2/files-2.1/com.android.tools.build/gradle
```

**Win**

```java
C:/Users/用户/.gradle/caches/modules-2/files-2.1/com.android.tools.build/gradle
```

**还有可能出现在:**

**MAC**

```
‎⁨硬盘⁩ ▸ ⁨应用程序⁩ ▸ ⁨Android Studio.app⁩ ▸ ⁨Contents⁩ ▸ ⁨gradle⁩ ▸ ⁨m2repository⁩ ▸ ⁨com⁩ ▸ ⁨android⁩ ▸ ⁨tools⁩ ▸ ⁨build⁩ ▸ ⁨gradle⁩ ▸ ⁨4.0.0⁩
```

**Win**

```
AS安装目录/⁨gradle⁩/⁨m2repository⁩/⁨com⁩/⁨android⁩/⁨tools⁩/build⁩/gradle⁩/⁨4.0.0⁩
```

> 注意:工程不需要能`run`, 只需要`sync`操作, 然后看日志输出即可.

# 开始分析

分析只有一个目标: 找到相关类, 找到相关类中的相关变量, 修改变量, 完成目标.

## 找到相关类

# 关键类1 BaseAppModuleExtension

```groovy
android {
   println it.class //class com.android.build.gradle.internal.dsl.BaseAppModuleExtension_Decorated
}
```

查看继承关系, 以及对应源码找到:`AbstractAppExtension`类的`val applicationVariants:DomainObjectSet<ApplicationVariant>`成员.

# 关键类2 ApplicationVariantImpl

```groovy
applicationVariants.all { variant ->
    println variant.class //com.android.build.gradle.internal.api.ApplicationVariantImpl_Decorated
}
```

查看继承关系, 以及对应源码找到:`ComponentProperties`类的`val outputs:List<VariantOutput>`成员.

# 关键类3 ApkVariantOutputImpl

```groovy
applicationVariants.all { variant ->
    variant.outputs.forEach{
        println it.class //com.android.build.gradle.internal.api.ApkVariantOutputImpl_Decorated
    }
}
```

查看继承关系, 以及对应源码找到:`BaseVariantOutputImpl`类的`protected final ApkData apkData`成员.

# 关键类4 ApkData

查看类`ApkData`, 引入眼前的`private String outputFileName;`成员.试一试, 应该就是它了.

```groovy
applicationVariants.all { variant ->
    variant.outputs.forEach{
        it.apkData.outputFileName = "test.apk"
    }
}
```

事实证明,成功了.

# 完整脚本

```groovy
android {
    applicationVariants.all { variant ->
        if (variant.buildType.name != "debug") {
            variant.packageApplicationProvider.get().outputDirectory = rootProject.file("/apk")
        }
        variant.outputs.forEach {
            it.apkData.outputFileName = "test.apk"
        }
    }
}
```

# 其他版本

[Android_Gradle_DSL_3.5](https://github.com/angcyo/Android_Gradle_DSL_3.5)

[Android_Gradle_DSL_3.3](https://github.com/angcyo/Android_Gradle_DSL_3.3)

[Android_Gradle_DSL_3.2](https://github.com/angcyo/Android_Gradle_DSL_3.2)