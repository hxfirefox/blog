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
Each node in the graph represents a module in a multi-module build, the "levels" simply indicate the distance to the first module in the internal reactor dependency graph. Maven calculates this graph based on declared inter-module dependencies for a multi-module build. Note that the parent maven project is also a dependency, which explains why there is a single node on top of most project graphs. Dependencies outside the reactor do not influence this graph.
For simplicity; let's assume all modules have an equal running time. This build should have level 0 running first, then a fanout of up to 5 parallel on level 1. On level 2 you'll be running 3 parallel modules, and 7 on 3, 5 on level 4.
This goes by declared dependencies in the pom, and there is no good log of how this graph is actually evaluated. (I was hoping to render the actual execution graph, but never got around to finding a cool tool/way to do it - plaintext ascii in the -X log would be one option).
Of course, in real life your modules do not take equal amounts of time. Significant gains are common when the project has one or more "api" modules and dependencies on the "api" modules (and just bring the "impl" version of the module into the actual assembly that will be started). This design normally means your big chunky modules depend on lightweight "api" modules that build quickly.
The parallel build feature rewards "correct" modularizations. If your project has degenerated inter-module dependencies (execessive dependencies inside reactor), you will probably see gains by cleaning up the dependencies.

从上述分析可以看出，依赖是maven能够正确识别构建先后顺序的，即使是并行构建场景。这种依赖关系处理可以有两种方式，视构建认为的规模而定。

- 解耦构建与打包，将打包从构建中剥离出来，单独执行
- 增强依赖，特别是在负责打包的部分，对于打包部分，要求显式地依赖其他的构建结构
