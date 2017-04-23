使用Sonar分析Java项目中的单元测试代码覆盖
======================================

# 概述
SonarQube可以导入测试执行和代码覆盖报告

# 准备工作
Sonar插件可以重用已经生成的测试报告，因此只需要保证测试报告在分析前正确生成就可以导入这些测试报告
- 按JUnit XML格式的测试执行报告
- Cobertura或JaCoCo生成的代码覆盖报告

# 用法

通过下列参数将测试执行和代码覆盖报告的路径传递给SonarQube

|属性|范围|范例|说明|
|:---|:--|:---|:---|
|sonar.junit.reportsPath|Project-wide|target/surefire-reports|导入测试执行报告(Surefire XML格式)，设置该属性为包含所有XML报告的文件路径|
|sonar.jacoco.reportPaths|Project-wide|target/jacoco.exec, target/jacoco-it.exec (default)|SonarQube 6.2以上版本支持，导入JaCoCo代码覆盖报告，设置该属性为JaCoCo .exec 报告路径，合并多份报告(SonarQube版本低于6.2，请使用sonar.jacoco.reportPath属性)|
