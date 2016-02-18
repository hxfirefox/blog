解决Maven并行编译中出现打包错误问题的思路
=========================================

# 并行构建

Maven 3.x 提供了并行编译的能力，通过执行下列命令就可以利用构建服务器的多线程/多核性能提升构建速度：

```
mvn -T 4 clean install # Builds with 4 threads
mvn -T 1C clean install # 1 thread per cpu core
mvn -T 1.5C clean install # 1.5 thread per cpu core
```
采用并行构建时，Maven会分析项目的依赖并规划出可以进行并行构建的模块

# 问题

通过并行编译确实能提升构建速度，但是如果在写pom时不加注意，那么很容易就会引发新问题。最近遇到并行编译引发打包错误的问题，问题的外在表现都很相似，即在使用maven插件进行打包时，发生了无法找到相应的包导致构建无效版本或无法产生打包版本的错误。

通常在一个项目中，想要子系统或模块实现先后依赖顺序，可以采用在parent pom中按照依赖顺序编写module的方式，如下所示。

```
<modules>
  <module>A</module>
  <module>B</module>
  <module>C</module>
</modules>
```

按照上述内如，A、B、C三个子模块将按照先A，再B，最后C的顺序依次构建，在串行构建时，情况确实如此，但是到了并行构建中，这样的顺序就无法保证了，即使能够保证按照A、B、C的顺序并行构建，由于子模块构建时间不同，依然会出现最后的子模块率先构建完成的情景。

# 解决之道

先回到maven看下它是如何处理构建，如下图所示。

![img](https://cwiki.apache.org/confluence/download/attachments/18153538/PastedGraphic-6.png?version=1&modificationDate=1379608014000&api=v2)
图中每个节点代表多模块构建中的单个模块，而“级别”标注了与第一个模块的在依赖关系上的距离，Maven根据模块依赖来计算出上图，需要注意的是maven中的继承关系也是一种依赖。
简化说明，假设所有的模块具有相等的构建时间，构建先从级别0开始，然后是并行的5个级别1；在级别2，会并行构建3个模块，以此类推，是级别3中个7个并行构建模块，这样构建方式源于pom中声明的依赖，当然在实际的构建过程中，各个模块的构建的时间是不一致，从而导致了上述问题

从上述分析可以看出，依赖是maven能够正确识别构建先后顺序的，即使是并行构建场景。这种依赖关系处理可以有两种方式，视构建认为的规模而定。

- 解耦构建与打包，将打包从构建中剥离出来，单独执行
- 增强依赖，特别是在负责打包的部分，对于打包部分，要求显式地依赖其他的构建结构

#参考

[Parallel builds in Maven 3](https://cwiki.apache.org/confluence/display/MAVEN/Parallel+builds+in+Maven+3)
