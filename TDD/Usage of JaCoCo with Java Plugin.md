Usage of JaCoCo with Java Plugin
================================

To launch JaCoCo as part of your Maven build, use this command: mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true
For more details on JaCoCo, see its documentation.
You may also find the README.md of the sample project helpful.
There is a known issue on version 0.7.3 of JaCoCo agent producing binary reports incompatible with JaCoCo analyzer 0.7.2 embedded in Java plugin. This issue is fixed with version 0.7.4 of the agent, so please prefer this version.
Java Plugin is compatible with JaCoCo 0.7.5 starting from Java Plugin 3.4 : 

Using argLine
If your project uses the argLine property to configure the surefire-maven-plugin, be sure that argLine defined as a property, rather than as part of the plugin configuration. Doing so will allow JaCoCo to set its agent properly. Otherwise the JVM may crash while tests are running.
That is, argLine should be defined this way: 

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
