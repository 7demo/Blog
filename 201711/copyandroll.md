>版本发布的时候，是采用的直接上传代码文件进行覆盖，操作之前需要先把之前的文件进行备份。同时还要考虑进行回滚的操作，因为写了两个脚本文件。

## 备份文件 copy.sh

```bash

#!/bin/bash

#先查看备份文件夹/srv/backup是否存在
#若不存在，在创建
# -d 文件夹是否存在
# -f 文件是否存在
# -x 是否有执行权限
# -n 判断一个变量是否有值
if [ ! -d /srv/backup ]
then
#创建文件夹
mkdir /srv/backup
fi

#定义将要备份的目录，即程序所在目录路径
aimdir='/srv/www/nodeapp/'
#获取目标/程序目录下的所有文件
aimdirls=`ls /srv/www/nodeapp`
#程序目录下所有的存放到的数组
filelist=()
#将要备份文件的之前的所有备份版本
backuplist=()

#输出
# 参数 -e 是解释反斜杠ESC(ASCII--\033)
# -E 关闭解释反斜杠ESC
# [背景色;字体颜色;动作（粗体、闪烁...）
echo '--------------------------'
echo -e '\033[0;33;1m-------选择备份-----------\033[0m '
echo '--------------------------'

#程序目录下所有的存放到的数组
for file in ${aimdirls}
do
#获取当前数组长度
len=${#filelist[@]}
#设置把文件推进数组
filelist[len]=$file
echo -e "-----\033[0;35;1m ${len}) ${file} \033[0m"
done

echo '--------------------------'
echo '--------------------------'

#选择要备份的文件输入
read -p "(选择备份的序号：):" index
echo $index
#根据选择的文件，清理相对应的冗余的备份文件（大于7个）
function clearCopyFiles(){
#拿到函数参数，赋值到变量，该变量是 将要备份的文件
filename=$1
backupfiles=()
#已经备份的所有文件
for file in `ls -t /srv/backup`
do
#去掉文件名称中的数字与-
# tr -d 表示删除
# tr -c 替换
# tr -s 去重
# echo 输出可以保存为字符串
splitfile=`echo "${file}" | tr -d [0-9] | tr -d "-"`
echo "${splitfile}*********${file}"
#只保存所有备份的文件中与要备份的文件一样，否则则跳过继续
if [ "$splitfile" != "$filename" ]
then
continue
fi
len=${#backupfiles[@]}
echo "====================${splitfile}, ${filename}"
backupfiles[len]=$file
done

len=${#backupfiles[@]}
echo "关于${filename}已备份的文件有${len}个=============${backupfiles[${len}-1]}"
#如果备份文件已经超过7个则删除
# -gt 大于
# -lt 小于
# -eq 等于
# -ne 不等于
# -ge 大于等于
# -le 小于
if [ $len -ge 5 ]
then
find /srv/backup/${backupfiles[0]} -delete
fi
}

#将要备份的文件
copyfile=${filelist[${index}]}

#清理冗余的备份文件，执行函数，第二个为参数
clearCopyFiles $copyfile

#对将要备份的文件进行命名
nowtime=`date +%y%m%d%H%M%S`
curname=${copyfile}-${nowtime}

#备份
cp -r ${aimdir}${copyfile} /srv/backup/${curname}
echo -e "\033[32m备份$copyfile成 ${curname} 成功\033[0m"

```

## 回滚文件 rollback.sh

```bash
#!/bin/bash

#程序所在目录
aimdir='/srv/www/nodeapp/'
#程序目录下的所有文件
aimdirls=`ls /srv/www/nodeapp`
#程序目录下的所有文件 将要存放的数组
filelist=()

#已经备份的文件的所有版本
backupfiles=()

echo '-----------------------'
echo -e '\033[0;33;1m-----请选择将要恢复的文件-----\033[0m'
echo '-----------------------'

for file in ${aimdirls}
do
len=${#filelist[@]}
filelist[len]=$file
echo -e "----\033[0;35;1m ${len}) ${file} \033[0m"
done

echo '-----------------------'
echo '-----------------------'

#选择要回滚的文件输入
read -p "(请选择备份的序号：):" index
rollbackfile=${filelist[$index]}
echo "---查找${rollbackfile}对应的所有版本"
#根据选择的文件查找对应的所有已备份的文件

function checkFiles() {
filename=$1
#根据名字查找已经对应的所有文件
echo -e "\033[0;33;1m --------请选择对应的版本-----------\033[m"
for file in `ls -lt /srv/backup`
do
splitfile=`echo "${file}" | tr -d [0-9] | tr -d "-"`
#如果改文件与选择的文件的主名字相等，则保存
if [ "$filename" != "$splitfile" ]
then
continue
fi
echo "--------------------> ${splitfile}"
len=${#backupfiles[@]}
backupfiles[$len]=$file
echo -e "======\033[0;34;1m ${len}),${file} \033[0m"

done
}

checkFiles $rollbackfile

#关于选择版本的处理
len=${#backupfiles[@]}
echo "关于${filename}的备份共${len}个"
echo "-----------------------------"
read -p "请选择备份的版本: " vindex
echo $vindex

#回滚版本
rollversion=${backupfiles[${vindex}]}
echo "&&&& ${rollversion} ${rollbackfile}"

# 根据回滚的是 文件 还是文件夹，做不同的区分
if [ -d ${aimdir}${rollbackfile} ]
then
echo '文件夹'
\cp -r /srv/backup/${rollversion}/* ${aimdir}${rollbackfile}
fi

if [ -f ${aimdir}${rollbackfile} ]
then
echo '文件'
\cp -r /srv/backup/${rollversion} ${aimdir}${rollbackfile}
fi
echo -e "\033[32m 回滚成功 \033[0m"


```


## 执行

要给予`sh`执行权限
`chmod +x copy.sh `
执行：
./copy.sh