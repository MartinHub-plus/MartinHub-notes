## 集群执行普通命令脚本 - Xcall

``` shell
#!/bin/bash
#在集群的所有机器上批量执行同一条命令
if(($#==0))
then
echo 请输入您要操作的命令！
exit
fi
echo 要执行的命令是$*
#循环执行此命令
for((i=101;i<=103;i++))
do
echo ---------------------hadoop$i-----------------
ssh hadoop$i $*
done
```

## 集群传输文件命令脚本 - Xsync

``` shell
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
echo Not Enough Arguement!
exit;
fi
#2. 遍历所有目录，挨个发送
for file in $@
do
#3. 获取父目录
pdir=$(cd -P $(dirname $file); pwd)

#4. 获取当前文件的名称
fname=$(basename $file)

#5. 遍历集群所有机器，拷贝
for host in hadoop102 hadoop103 hadoop104
do
echo ====================    $host    ====================
rsync -av $pdir/$fname $USER@$host:$pdir
done
done
```

## 集群Jps脚本 - Jps

```shell
#!/bin/bash
for i in hadoop102 hadoop103 hadoop104
do
	echo =====================  $i  =====================
	ssh $i "source /etc/profile && jps $@ | grep -v Jps"
done
```

## 检测进程-服务死掉后重启脚本 -Restart

```shell
#!/bin/bash
echo '========================== 自动检测服务程序 ==============================='
while :
do
nginx_count=`ps -ef | grep nginx | grep -v grep | wc -l`
hive_count=`ps -ef | grep hive | grep -v grep | wc -l`
kafka_count=`ps -ef | grep kafka | grep -v grep | wc -l`
hbase_count=`ps -ef | grep hbase | grep -v grep | wc -l`

if [ $nginx_count -eq 0 ];then
echo '>>>>>>>>>>>>>>> Nginx 服务已挂掉，正在重启服务！！！！！！！'
`/usr/local/webserver/nginx/sbin/nginx`
echo '>>>>>>>>>>>>>>> Nginx 服务已成功重启！！！！！！！'
fi

if [ $hive_count -eq 0 ];then
echo '>>>>>>>>>>>>>>> hive 服务已挂掉，正在重启服务！！！！！！！'
`/opt/hadoop/hive-3.1.0/bin/hive --service metastore`
`/opt/hadoop/hive-3.1.0/bin/hive --service hiveserver2`
echo '>>>>>>>>>>>>>>> hive 服务已成功重启！！！！！！！'
fi

if [ $kafka_count -eq 0 ];then
echo '>>>>>>>>>>>>>>> kafka 服务已挂掉，正在重启服务！！！！！！！'
`/opt/hadoop/kafka-1.1.1/bin/kafka-start.sh`
echo '>>>>>>>>>>>>>>> kafka 服务已成功重启！！！！！！！'
fi

if [ $hbase_count -eq 0 ];then
echo '>>>>>>>>>>>>>>> hbase 服务已挂掉，正在重启服务！！！！！！！'
`/opt/hadoop/hbase-2.1.7/bin/hbase-daemon.sh start master`
`/opt/hadoop/hbase-2.1.7/bin/hbase-daemon.sh start regionserver`
echo '>>>>>>>>>>>>>>> hbase 服务已成功重启！！！！！！！'
fi

done
```



