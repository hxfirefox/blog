迁移SVN托管项目至gerrit记录
=========================

# 迁移的原因
项目在回顾研发过程中总结了以下痛点：

**痛点1：代码分支易受污染**

开发人员采用流程基本如下：
- 1.修改代码
- 2.提交代码
- 3.构建
- 4.集成版本
- 5.执行FT
- 6.定位发现问题
- 7.修改代码...

上述流程就意味着一旦发生错误合入或者漏合入，对主分支造成污染，并导致版本在一定时间内不可用

**痛点2：持续集成反馈周期长，影响追溯困难**

上述研发流程执行一次通常需要2～3小时，一旦发现合入代码存在问题时，则会恶化执行流程的耗时，进而导致开发人员不等待反馈结果就会执行其他修改代码工作（重复上述流程），而当反馈结果存在问题时，就需要追溯多个修改

**痛点3：版本构建/升级困难**

版本构建的相互依赖更加加剧了长反馈周期，此外目前版本的升级工作繁琐、操作步骤多、波及的团队广，导致版本稳定周期长，要求每个团队都参与其中，但实际上业务团队对于版本升级应尽量保持不感知，除非新特性在业务中得到使用

# 解决思路
**引入试飞中心**

针对痛点1、2，改进手段是让代码在合入主分支前进行验证和审核，借助gerrit使得提交的代码首先进行编译、测试后再合入主分支，这些过程完全自动化进行，开发者只感知代码审核结果即可，从而保证了主分支代码的纯净

**引入聚合项目**

针对痛点3，改进手段是引入聚合项目即单独建立一个项目，此项目包含其他组件项目（使用git submodule），通过依赖管理，如pom，管理各个组件项目使得版本构建简化；并通过自动化脚本统一修改所有组件版本从而简化升级过程，而无需像现在这样需要多个团队进行长时间协作

# 待办事项
从上面的痛点分析、解决思路不难看出，实施项目持续集成改造需要完成以下几个操作：
- 迁移svn代码库至git
- 迁移过程中对代码进行解耦，形成可独立发布的repo
- gerrit结合jenkins完成试飞中心
- 聚合项目

与项目讨论后，又补充一条：
- 保留svn操作日志

# 迁移
利用git svn指令可以轻松地完成迁移，并能够保留历史记录，即迁移svn log至git log，如果想做的更完美就需要首选转换author信息，在 svn中，每个提交者拥有一个用户名，记录在提交信息中，在本文服务的项目中author信息格式为[用户名][ID]，为了让这条信息更好的映射到git author数据里，需要建立一个相互间的映射关系。

首先，将svn log中的author信息保留下来，采用xml格式\<author\>用户名+ID\</author\>
```
svn log --xml [svn路径] | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > user.txt
```
接着，需要将author信息转换git中的映射，格式为[用户名][ID]=[ID]\<ID@email\>

```
#!/bin/bash

if [ -e "usertmp.txt" ]; then
     rm -rf usertmp.txt
fi

if [ -e "author.txt" ]; then
     rm -rf author.txt
fi

cat $1 | awk '{print}' | sed 's/<author>//g' | sed 's/<\/author>/=<@email>/g' | sed -e 's/ /\n/g' >> usertmp.txt
 
cat usertmp.txt | while read line
do
     id=(`echo ${line} | grep -o '[0-9]\+'`)
     echo ${line}| sed -e "s/</${id[0]}<${id[0]}/g" >> author.txt
done
```

有了上述信息，接下来的就是使用git svn clone指令

```
git svn clone [svn url] --trunk=[trunk subdir] --authors-file=author.txt --no-metadata -s [target dir]
```

在执行git svn clone时会遇到不少问题，最典型的是如下的错误**Ignoring error from SVN, path probably does not exist: (160013)**，发生此错误的原因是由于svn分支中并非所有的文件路径都是从最初阶段产生的，而是随着项目的变化逐个添加或者删除，而clone时则是从最初版本开始，解决这个问题可以略微修改指令，如下。并且在实际操作过程中，如果需要从svn分支下的某个文件路径衍生出git库来，这样做可以节省检出时间。

```
git svn clone -r[version1]:[version2] [svn url] --trunk=[trunk subdir] --authors-file=author.txt --no-metadata -s [target dir]
```

# 分割与合并
这次迁移活动涉及多地多团队，由于一些历史原因，之前svn库中，各个团队的代码耦合在一起，因此这次迁移正好也是一次调整代码目录解耦的机会。库的分割相对来说较为简单，svn基于目录的检出方式能够为库的分割提供较大的便利，而合并则稍微复杂一下，下面演示了如何将两个由svn迁移到git的库合并。

首先，分别检出两个git库，执行如下指令，通过git log可以查看到各自的日志也同时迁移至git中

```
git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=configuration --authors-file=userinfo.txt --no-metadata -s nfmconf

git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=southconfig --authors-file=usersc.txt --no-metadata -s nfm_sc
```

![img=logo1](https://github.com/hxfirefox/blog/blob/master/TDD/img/log1.png)

![img=logo2](https://github.com/hxfirefox/blog/blob/master/TDD/img/log2.png)

选定nfmconf作为master，然后将nfm_sc库作为前者的远程仓库，执行如下指令

```
git remote add other ../nfm_sc/
git fetch other
```

将fm_sc仓库的master分支作为新分支checkout到本地，新分支名设定为repo1

```
git checkout -b repo1 other/master
```

切换回master分支，并合并分支

```
git checkout master
git merge repo1
```

此时在查看git log就能够看到两个库的log已经合并到了一起

![img=logo3](https://github.com/hxfirefox/blog/blob/master/TDD/img/log3.png)

# 参考
在实施过程中参考了如下文章：

- [坑死人不偿命，svn 迁移到 git](http://blog.csdn.net/cctt_1/article/details/41317419)
- [SVN 迁移到 Git](http://blog.csdn.net/lhzhang1985/article/details/6294223)
- [合并两个git仓库](http://blog.csdn.net/gouboft/article/details/8450696)
