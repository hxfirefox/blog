如何在Maven和Gradle中配置使用Groovy 2.4与Spock 1.0
======================================================================

>原文 https://dzone.com/articles/spock-10-groovy-24
>
>翻译 hxfirefox

#Maven

Maven无法天然支持除Java外的其他JVM语言，例如Groovy或Scala。想要在Maven中使用Groovy/Spock，需要引入第三方插件。对于Groovy，最佳选择是GMavenPlus，另一个选择是使用Groovy-Eclipse编译器的插件，不过这种插件不使用官方groovyc，并且当Groovy发布新特性时会存在问题。

使用GMavenPlus插件的配置范例如下：

```
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>1.4</version>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>testCompile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
使用Spock编写的测试文件建议用Spec后缀，此外还需通知Surefire需要查询的测试文件，如下所示：

```
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${surefire.version}</version>
    <configuration>
        <includes>
            <include>**/*Spec.java</include> <!-- Yes, .java extension -->
            <include>**/*Test.java</include> <!-- Just in case of having also "normal" JUnit tests -->
        </includes>
    </configuration>
</plugin>
```
注意此处需要包括 \*\*/\*Spec.java 而不是 \*\*/\*Spec.groovy，此外在依赖中需要增加如下：

```
<dependencies>
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>2.4.1</version>
    </dependency>
    <dependency>
        <groupId>org.spockframework</groupId>
        <artifactId>spock-core</artifactId>
        <version>1.0-groovy-2.4</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```
使用合适的Spock版本非常重要，对于Groovy 2.4，应该选择版本1.0-groovy-2.4的GMavenPlus；对于Groovy 2.3，则应选择1.0-groovy-2.3。一旦选错了版本，就会出现如下信息警告：

```
Could not instantiate global transform class
org.spockframework.compiler.SpockTransform specified at
jar:file:/home/foo/.../spock-core-1.0-groovy-2.3.jar!/META-INF/services/org.codehaus.groovy.transform.ASTTransformation
because of exception
org.spockframework.util.IncompatibleGroovyVersionException:
The Spock compiler plugin cannot execute because Spock 1.0.0-groovy-2.3 is
not compatible with Groovy 2.4.0. For more information, see
http://versioninfo.spockframework.org
```
结合其他pom中的元素，上述pom就会超过50行XML，其中大部分与Groovy和Spock相关

#Gradle

Gradle内建支持Groovy和Scala，只需要应用Groovy插件，如下所示：

```
apply plugin: 'groovy'
```
接着添加依赖：

```
compile 'org.codehaus.groovy:groovy-all:2.4.1'
testCompile 'org.spockframework:spock-core:1.0-groovy-2.4'
```
以及Gradle查询上述信息的仓储：

```
repositories {
    mavenCentral()
}
```
结合定义包组和版本部分，采用基于Groovy的DSL编写的Gradle总共约为15行。对于Gradle，Spock和Groovy的版本同样很重要，例如，Groovy 2.4.1与Spock 1.0-groovy-2.4。

#Summary

从上面的对比不难看出，在使用Spock时，Gradle是更加出色的解决方案。下图示出了Spock与Groovy配置在Maven和Gradle中比较：

![img](https://solidsoft.files.wordpress.com/2015/03/spock-groovy-maven-gradle.png)
