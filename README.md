# Maven项目 子父工程、依赖引用情况下组件漏洞修复测试

## 工程结构

maven-demo ------xstream ？

  └── demo-app

  └── demo-sdk

     └── xstream ？

依赖方式
demo-app依赖demo-sdk，所以demo-sdk会最先打包，demo-app会最后打包

## 子工程demo-sdk测试
1、当主pom maven-demo的pom在dependencyManagement定义xstream为1.4.9危险版，子pom demo-sdk引用xstream但不定义版本，此时会使用父工程中的
maven-demo
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.thoughtworks.xstream</groupId>
                <artifactId>xstream</artifactId>
                <version>1.4.9</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
demo-sdk
```
    <dependencies>
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
        </dependency>
    </dependencies>
```
进入demo-sdk目录 运行 mvn dependency:tree
```
[INFO] com.example:demo-sdk:jar:1.0-SNAPSHOT
[INFO] \- com.thoughtworks.xstream:xstream:jar:1.4.9:compile
[INFO]    +- xmlpull:xmlpull:jar:1.1.3.1:compile
[INFO]    \- xpp3:xpp3_min:jar:1.1.4c:compile
```
2、当主pom maven-demo的pom在dependencyManagement定义xstream为1.4.9危险版，子pom demo-sdk引用xstream定义1.4.21版本。
只需要将上述demo-sdk改为
```
    <dependencies>
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>1.4.21</version>
        </dependency>
    </dependencies>
```
进入demo-sdk目录 运行 mvn dependency:tree 选的是1.4.21版本，因子pom可以覆盖
```
[INFO] com.example:demo-sdk:jar:1.0-SNAPSHOT
[INFO] \- com.thoughtworks.xstream:xstream:jar:1.4.21:compile
[INFO]    \- io.github.x-stream:mxparser:jar:1.2.2:compile
[INFO]       \- xmlpull:xmlpull:jar:1.1.3.1:compile
```

## 整个项目测试，即demo-app项目测试，因为根据依赖关系最终打包的是demo-app项目
1.demo-app只引用demo-sdk，同时不使用dependencyManagement和dependency去定义xstream版本，demo-sdk显引用xstream 1.4.21，maven-demo在dependencyManagement定义xstream为1.4.9
demo-app 
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>demo-root</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>demo-app</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>demo-sdk</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```
demo-sdk
```
    <dependencies>
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>1.4.21</version>
        </dependency>
    </dependencies>
```
maven-demo
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.thoughtworks.xstream</groupId>
                <artifactId>xstream</artifactId>
                <version>1.4.9</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
在maven-demo目录下执行mvn dependency:tree，显示xstream为1.4.9，虽然demo-sdk传递的是1.4.21版本，但是demo-app未定义xstream版本，对于demo-ap来说父级pom继承大于依赖传递，所以demo-app用的是maven-demo在dependencyManagement定义的1.4.9版本
```
[INFO] com.example:demo-app:jar:1.0-SNAPSHOT
[INFO] \- com.example:demo-sdk:jar:1.0-SNAPSHOT:compile
[INFO]    \- com.thoughtworks.xstream:xstream:jar:1.4.9:compile
[INFO]       +- xmlpull:xmlpull:jar:1.1.3.1:compile
[INFO]       \- xpp3:xpp3_min:jar:1.1.4c:compile
```
2.maven-demo和demo-app同时不使用dependencyManagement和dependency去定义xstream版本，demo-sdk显引用xstream 1.4.21
只需要将上面的maven-demo的pom中的dependencyManagement相关代码注释掉

在maven-demo目录下执行mvn dependency:tree，显示xstream为1.4.21，直接依赖传递
```
[INFO] com.example:demo-app:jar:1.0-SNAPSHOT
[INFO] \- com.example:demo-sdk:jar:1.0-SNAPSHOT:compile
[INFO]    \- com.thoughtworks.xstream:xstream:jar:1.4.21:compile
[INFO]       \- io.github.x-stream:mxparser:jar:1.2.2:compile
[INFO]          \- xmlpull:xmlpull:jar:1.1.3.1:compile
```
## 上述修复方式
1、父工程的dependencyManagement直接定义安全版本
maven-demo
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.thoughtworks.xstream</groupId>
                <artifactId>xstream</artifactId>
                <version>1.4.21</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
2、最终打包的demo-app直接定义一个安全的dependencyManagement
demo-app
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.thoughtworks.xstream</groupId>
                <artifactId>xstream</artifactId>
                <version>1.4.21</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
3、最终打包的demo-app直接定义一个安全的dependency
demo-app
```
    <dependencies>
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>1.4.21</version>
        </dependency>
    </dependencies>
```
上述3种情况在maven-demo目录下执行mvn dependency:tree，显示xstream为1.4.21
```
[INFO] com.example:demo-app:jar:1.0-SNAPSHOT
[INFO] \- com.example:demo-sdk:jar:1.0-SNAPSHOT:compile
[INFO]    \- com.thoughtworks.xstream:xstream:jar:1.4.21:compile
[INFO]       \- io.github.x-stream:mxparser:jar:1.2.2:compile
[INFO]          \- xmlpull:xmlpull:jar:1.1.3.1:compile
```
## 结论
1.子pom不应该显示声明版本，非常危险，子pom的组件版本统一由主Pom的dependencyManagent管理。

2.子pom如需设置组件版本，必须设为安全版本。

3. 依赖情况下最终SpringBoot打包工程也需要一份安全的dependencyManagent管理版本来限制最终的间接依赖，防止其引入的危险组件同时父工程pom中的dependencyManagent遗漏版本定义或定义了危险版本的情况。
