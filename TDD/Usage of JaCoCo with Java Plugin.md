如何使用JaCoCo的Java插件
=======================

使用如下命令可以在maven构建过程中启动：

```
mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true
```

JaCoCo 0.7.3产生的二进制报告与JaCoCo分析器0.7.2无法兼容，这是一个已知的问题，并且在0.7.4中得到修复。从Java插件3.4开始，兼容JaCoCo 0.7.5。

**使用argLine**

如果项目使用了argLine属性来配置surefire-maven-plugin，应将argLine定义成property，而不是插件配置的一部分。这样做可以让JaCoCo正确地设置代理，否则当测试运行时可能会发生JVM崩溃。

```
      <properties>
        <argLine>-Xmx128m</argLine>
      </properties>
...
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
          <runOrder>random</runOrder>
        </configuration>
      </plugin>
```

**强制覆盖率为0%**

缺省情况下，没有覆盖率报告时，插件会设置任意值，通过下列属性可以在无报告情况下，强制覆盖率为0%。 

```
sonar.jacoco.reportMissing.force.zero=true
```

**每个测试的覆盖率** 

通过使用一些UT数据搜集工具，可以获得测试的覆盖情况，可以参考范例项目中的pom.xml
