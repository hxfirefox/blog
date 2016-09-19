如何迁移SVN托管项目至git
========================

参考：


svn log --xml | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > user.txt
// TODO: 用脚本转换xml为[author]=[author]<email>

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

git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=zxr10-models --authors-file=user.txt --no-metadata -s nfmdevmod
移动使用mv，移动后应加上.gitignore和pom.xml
svn log --xml http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic/configuration | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > userinfo.txt
git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=configuration --authors-file=userinfo.txt --no-metadata -s nfmconf
