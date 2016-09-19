参考：


svn log --xml | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > user.txt
// TODO: 用脚本转换xml为[author]=[author]<email>
#!/bin/bash
      2 
      3 if [ -e "usertmp.txt" ]; then
      4     rm -rf usertmp.txt
      5 fi
      6 
      7 if [ -e "author.txt" ]; then
      8     rm -rf author.txt
      9 fi
     10 
     11 cat $1 | awk '{print}' | sed 's/<author>//g' | sed 's/<\/author>/=<@zte.com.cn>/g' | sed -e 's/ /\n/g' >> usertmp.txt
     12 
     13 cat usertmp.txt | while read line
     14 do
     15     a=(`echo ${line} | grep -o '[0-9]\+'`)
     16     echo ${line}| sed -e "s/</${a[0]}<${a[0]}/g" >> author.txt
     17 done
git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=zxr10-models --authors-file=user.txt --no-metadata -s nfmdevmod
移动使用mv，移动后应加上.gitignore和pom.xml
svn log --xml http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic/configuration | grep author | sort -u | perl -pe 's/.>(.?)<./$1 - /' | xargs > userinfo.txt
git svn clone http://10.75.8.170/svn/ZXNFM_REPOS/trunk/SDN_NFManager/50.zenic --trunk=configuration --authors-file=userinfo.txt --no-metadata -s nfmconf
