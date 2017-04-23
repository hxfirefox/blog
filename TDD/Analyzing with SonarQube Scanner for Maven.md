基于Maven的Sonar分析
===================

# 特性

建议在基于Maven管理的Java项目上进行本文所述内容

# 兼容性

| Maven Version | 2.x | 3.x |
|:--------------|:----|:----|
|Compatibility	|  N	|  Y  |
 
从maven-sonar-plugin 2.7版本开始，就不再支持低于4.5的SonarQube版本，因此如果SonarQube版本低于4.5，应使用maven-sonar-plugin 2.6版本。
从maven-sonar-plugin 3.1版本开始，就不再支持低于3.0的Maven < 3.0版本，因此如果Maven版本低于3.0，应使用maven-sonar-plugin 3.0.2版本。

# 准备工作

需要具备如下软件及环境：
- Maven 3.x
- 安装SonarQube
- 安装的SonarQube服务支持的最低Java版本
- 分析各种编码语言的插件
- 已经阅读“Analyzing Code Source” 

# 首次安装
## 全局设置

编辑位于$MAVEN_HOME/conf或~/.m2路径下的settings.xml文件，设置插件前缀以及可选的SonarQube服务器地址，范例如下：
```
<settings>
    <pluginGroups>
        <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
    </pluginGroups>
    <profiles>
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <!-- Optional URL to server. Default value is http://localhost:9000 -->
                <sonar.host.url>
                  http://myserver:9000
                </sonar.host.url>
            </properties>
        </profile>
     </profiles>
</settings>
```

# 分析Maven项目

分析Maven工程需要在工程pomx.xml所在路径下执行Maven命令：sonar:sonar
```
mvn clean verify sonar:sonar
 
# In some situation you may want to run sonar:sonar goal as a dedicated step. Be sure to use install as first step for multi-module projects
mvn clean install
mvn sonar:sonar
 
# Specify the version of sonar-maven-plugin instead of using the latest. See also 'How to Fix Version of Maven Plugin' below.
mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar
```

要想获得测试覆盖信息，需要在分析执行前产生测试覆盖报告。想要获取更多信息请查看[Code Coverage by Unit Tests for Java Project and Code Coverage by Integration Tests for Java Project](https://docs.sonarqube.org/display/PLUG/Code+Coverage+by+Unit+Tests+for+Java+Project)

## 配置SonarQube分析

pom.xml文件样例如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.sonarqube</groupId>
  <artifactId>example-java-maven</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>Java :: Simple Project :: SonarQube Scanner for Maven</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <sonar.language>java</sonar.language>
  </properties>

</project>
```
如需要更多的分析参数可以在[the Analysis Parameters](https://docs.sonarqube.org/display/SONAR/Analysis+Parameters)中找到

**安全**

通过sonar.login属性可以提供用户执行分析权限，范例如下：sonar-scanner -Dsonar.login=[my analysis token]

## 从分析中排除无需分析的模块

- 可以通过在想要排除的模块的pom.xml文件中定义属性<sonar.skip>true</sonar.skip>或者
- 使用构建规则来排除某些模块（类似集成测试）
- 使用Advanced Reactor选项（如 "-pl"），例如：mvn sonar:sonar -pl !module2

## 样例项目

To help you get started, a simple project sample is available on github that can be browsed or downloaded: projects/languages/java/maven/java-maven-simple
How to Fix Version of Maven Plugin
It is recommended to lock down versions of Maven plugins:

**roject analyzed with Maven 3**

```
<build>
  <pluginManagement>
    <plugins>
      <plugin>
        <groupId>org.sonarsource.scanner.maven</groupId>
        <artifactId>sonar-maven-plugin</artifactId>
        <version>3.2</version>
      </plugin>
    </plugins>
  </pluginManagement>
</build>
```

# 故障定位

