如何使用JaCoCo的Java插件
=======================

使用如下命令可以在maven构建过程中启动：

```
mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true
```

JaCoCo 0.7.3产生的二进制报告与JaCoCo分析器0.7.2无法兼容，这是一个已知的问题，并且在0.7.4中得到修复。
Java Plugin is compatible with JaCoCo 0.7.5 starting from Java Plugin 3.4 : 

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

Force coverage to 0%
By default, when no coverage report is found, the Java Plugin will not set any value for coverage metric. This behavior can be overridden to force coverage to 0% in case of a lack of report by setting the following property : 

```
sonar.jacoco.reportMissing.force.zero=true
```

If this property is set while a Jacoco report is provided, this property will have no effect.

#Coverage per test 

Using some unit test listeners you can collect the information on which lines where covered by which tests and display them in SonarQube. See the pom.xml of the sample project to configure this and see the Component Viewer documentation to display this information in SonarQube.
To launch JaCoCo as part of your Maven build, use this command: mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true
For more on JaCoCo, see its documentation.
You may also find the README.md of the sample project helpful.
