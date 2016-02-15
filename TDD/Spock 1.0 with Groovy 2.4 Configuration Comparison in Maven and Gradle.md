Spock 1.0 with Groovy 2.4 Configuration Comparison in Maven and Gradle
======================================================================

>原文 https://dzone.com/articles/spock-10-groovy-24
>
>翻译 hxfirefox

Spock 1.0 has been finally released. About new features and enhancements I already wrote two blog posts. One of the recent changes was a separation on artifacts designed for specific Groovy versions: 2.0, 2.2, 2.3 and 2.4 to minimize a chance to come across a binary incompatibility in runtime (in the past there were only versions for Groovy 1.8 and 2.0+). That was done suddenly and based on the messages on the mailing list it confused some people. After being twice asked to help properly configure two projects I decided to write a short post presenting how to configure Spock 1.0 with Groovy 2.4 in Maven and Gradle. It is also a great place to compare how much work is required to do it in those two very popular build systems.

#Maven

Maven does not natively support other JVM languages (like Groovy or Scala). To use it in the Maven project it is required to use a third party plugin. For Groovy the best option seems to be GMavenPlus (a rewrite of no longer maintained GMaven plugin). An alternative is a plugin which allows to use Groovy-Eclipse compiler with Maven, but it is not using official groovyc and in the past there were problems with being up-to-date with the new releases/features of Groovy.

Sample configuration of GMavenPlus plugin could look like:

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
As we want to write tests in Spock which recommends to name files with Spec suffix (from specification) in addition it is required to tell Surefire to look for tests also in those files:

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
Please notice that it is needed to include **/*Spec.java not **/*Spec.groovy to make it work.

Also dependencies have to be added:

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
It is very important to take a proper version of Spock. For Groovy 2.4 version 1.0-groovy-2.4 is required. For Groovy 2.3 version 1.0-groovy-2.3. In case of mistake Spock protests with a clear error message:

Could not instantiate global transform class
org.spockframework.compiler.SpockTransform specified at
jar:file:/home/foo/.../spock-core-1.0-groovy-2.3.jar!/META-INF/services/org.codehaus.groovy.transform.ASTTransformation
because of exception
org.spockframework.util.IncompatibleGroovyVersionException:
The Spock compiler plugin cannot execute because Spock 1.0.0-groovy-2.3 is
not compatible with Groovy 2.4.0. For more information, see
http://versioninfo.spockframework.org

Together with other mandatory pom.xml elements the file size increased to over 50 lines of XML. Quite much just for Groovy and Spock. Let’s see how complicated it is in Gradle.

#Gradle

Gradle has built-in support for Groovy and Scala. Without further ado Groovy plugin just has to be applied.

```
apply plugin: 'groovy'
```
Next the dependencies has to be added:

```
compile 'org.codehaus.groovy:groovy-all:2.4.1'
testCompile 'org.spockframework:spock-core:1.0-groovy-2.4'
```
and the information where Gradle should look for them:

```
repositories {
    mavenCentral()
}
```
Together with defining package group and version it took 15 lines of code in Groovy-based DSL.

Btw, in case of Gradle it is also very important to match Spock and Groovy version, e.g. Groovy 2.4.1 and Spock 1.0-groovy-2.4.

#Summary

Thanks to embedded support for Groovy and compact DSL Gradle is preferred solution to start playing with Spock (and Groovy in general). Nevertheless if you prefer Apache Maven with a help of GMavenPlus (and XML) it is also possible to build project tested with Spock.

The minimal working project with Spock 1.0 and Groovy 2.4 configured in Maven and Gradle can be cloned from my GitHub.

Graphical comparison of Spock and Groovy configuration in Maven and Gradle

[img](https://solidsoft.files.wordpress.com/2015/03/spock-groovy-maven-gradle.png)

**Note 1.** I haven’t been using Maven in my project for over 2 years (I prefer Gradle), so if there is a better/easier way to configure Groovy and Spock with Maven just let me know in the comments.

**Note 2.** The configuration examples assume that Groovy is used only for tests and the production code is written in Java. It is possible to mix Groovy and Java code together, but then the configuration is a little more complicated.
