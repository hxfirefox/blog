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
