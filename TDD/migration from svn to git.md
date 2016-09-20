如何迁移SVN托管项目至git
========================

# 背景

# 需求

# 实施

## 迁移
利用git svn指令可以轻松地完成迁移，并能够保留历史记录，即迁移svn log至git log，如果想做的更完美就需要首选转换author信息，在 svn中，每个提交者拥有一个用户名，记录在提交信息中，在本文服务的项目中author信息格式为[用户名][ID]，为了让这条信息更好的映射到git author数据里，需要建立一个相互间的映射关系。

首先，将svn log中的author信息保留下来，采用xml格式\<author\>用户名+ID\</author\>
```
svn log --xml [svn路径] | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > user.txt
```
接着，需要将author信息转换git中的映射，格式为[用户名][ID]=[ID]\<ID@zte.com.cn\>

```
#!/bin/bash

if [ -e "usertmp.txt" ]; then
     rm -rf usertmp.txt
fi

if [ -e "author.txt" ]; then
     rm -rf author.txt
fi

cat $1 | awk '{print}' | sed 's/<author>//g' | sed 's/<\/author>/=<@zte.com.cn>/g' | sed -e 's/ /\n/g' >> usertmp.txt
 
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

# 参考
在实施过程中参考了如下文章：

- [坑死人不偿命，svn 迁移到 git](http://blog.csdn.net/cctt_1/article/details/41317419)
- [SVN 迁移到 Git](http://blog.csdn.net/lhzhang1985/article/details/6294223)
- [合并两个git仓库](http://blog.csdn.net/gouboft/article/details/8450696)
