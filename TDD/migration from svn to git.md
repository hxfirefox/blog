如何迁移SVN托管项目至git
========================

# 背景

# 需求

# 实施 

svn log --xml [svn路径] | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > user.txt

转换xml为[author]=[author]\<email\>

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
```
git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=zxr10-models --authors-file=user.txt --no-metadata -s nfmdevmod
```

移动使用mv，移动后应加上.gitignore和pom.xml

```
svn log --xml http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic/configuration | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > userinfo.txt
git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=configuration --authors-file=userinfo.txt --no-metadata -s nfmconf
```

# 参考
在实施过程中参考了如下文章：

- [坑死人不偿命，svn 迁移到 git](http://blog.csdn.net/cctt_1/article/details/41317419)
- [SVN 迁移到 Git](http://blog.csdn.net/lhzhang1985/article/details/6294223)
- [合并两个git仓库](http://blog.csdn.net/gouboft/article/details/8450696)
