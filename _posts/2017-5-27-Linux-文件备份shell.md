---
title: Linux-文件备份shell
date: 2016-08-31 23:24:44
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390144   
  ==========================backup=======================================  
#!/bin/bash  
DIR="/tmp/test"  
BAK="$DIR/backup/"  
SRC="$DIR/source/"  
TODAY=$(date +"%F %H:%M:%S")  
FILELIST="$DIR/filelist.txt"  
TIME_FILE="$DIR/time_key"  
  
  
FULL () {  
touch $TIME_FILE  
find $SRC -type f > $FILELIST  
mkdir -p $BAK/"$TODAY"   
tar -T $FILELIST -czf - | tar -xzf - -C $BAK/"$TODAY"  
sed -i "s/$/\t$TODAY/" $FILELIST  
}  
  
  
UPDATE () {  
 touch $TIME_FILE.new  
 find $SRC -type f -newer $TIME_FILE > $DIR/updatefilelist.txt  
 mkdir -p $BAK/"$TODAY"  
 tar -T $DIR/updatefilelist.txt -czf - | tar -xzf - -C $BAK/"$TODAY"  
 /bin/mv $TIME_FILE.new $TIME_FILE  
 while read F_NAME  
 do   
STR=$(grep "^$F_NAME\>" $FILELIST)  
if [ -z "$STR" ]  
then  
echo -e "$F_NAME\t$TODAY" >> $FILELIST  
else  
sed -i "s%$STR%$F_NAME\t$TODAY%" $FILELIST  
fi  
 done < $DIR/updatefilelist.txt  
  
  
}  
case $1 in   
full)  
FULL;;  
update)  
UPDATE;;  
*)  
echo "bash $0 (full|update)";;  
esac   
 