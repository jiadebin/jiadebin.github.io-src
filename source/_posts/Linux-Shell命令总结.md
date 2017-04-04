---
title: Linux Shell命令总结
date: 2017-04-03 10:36:53
tags: [shell, Linux]
---

1. 使用find命令找到大于指定大小的文件：  
	find /-type f -size +10G
2. 在Linux下如何让文件夹下的文件让文件按大小排序？  
	方法一：# ls -lhSl 长格式显示，h human readable模式，大小单位为M,G等易读格式，S size按大小排序；  
    方法二：# du -h | sort -n
	当然您也可以结合管道查看文件夹内最大的几个文件或最小的几个文件, 再加上管道符号和head或者tail命令即可du -h | sort -n|head du -h * | sort -n|tail
3. 命令最后加上 & 符号表示在后台执行，shell不会阻塞；
4. rz 在xshell中接收文件
5. sz filename 在xshell中发送文件到windows主机
6. sudo su #切换到root shell
7. more /etc/issue #查看操作系统版本
8. 查找文件：
	find /path -name filename #系统内查找该文件(文件名必须是全的，不能部分匹配)  
	grep xxx path/* -nr #搜索path路径下包含xxx内容的文件(只查找文件)，n表示显示行号，r表示在path子目录中递归执行  
	locate filename  
9. [sudo] kill -9 进程号 #杀死进程，前面的sudo可选
10. ps –ef | grep 进程名 #查看包含进程名的进程，如 ps -ef | grep hive
11. lsof –i:端口号 #查看端口号占用情况
12. netstat –nltp | grep 进程号 #RedHat查看进程号所占用的端口号  
	netstat -an | grep 端口号 #查看端口占用情况  
13. sudo fuser 端口号/tcp #查看tcp端口占用的进程号
14. 组合：ps -ef | grep $(fuser 端口号/tcp)
15. netstat -anp | grep pid #ubuntu:查看进程占用端口号
16. sudo cat /etc/group #查看用户组
17. sudo cat /etc/passwd #查看所有用户
18. sudo useradd xxx
19. sudo userdel name
20. sudo usermod -a -G groupname username
21. sudo rpm -ivh xxx.rpm
22. sudo rpm -e xxx
23. sudo yum install xxx
24. sudo yum remove xxx
25. ln -s source/folder targetLinkName(不存在的一个名字) #注意删除软连接内容相当于删除了真实的文件
26. du -sh /folder1/ #查看folder1目录占用的总空间
27. du -sh /folder1/* #查看folder1目录里面各个子文件（夹）空间占用情况
28. df -lh #查看各个文件系统空间占用情况
29. tailf xxx.log #显示log文件最后十行内容，文件变时会自动更新，与 tail -f 命令的区别是log文件长度不变时不会自动读磁盘，故更省资源
30. sudo chkconfig mysqld on/off #设置/关闭 开机自启动
31. Ctrl r 进入reverse-i(incremental)-search 反向增量搜索模式，输入字母即可搜索出历史命令，按左右方向键开始编辑命令；
32. vi - #以stdin的内容为内容编辑一个新文件
33. grep xxx path/* -nr | vi - #即将查找到的行作为内容显示在一个新文件中，用vi打开
34. pdsh -R ssh -g groupname “ls -l” 在配置好的无密码登陆集群执行ls命令，注意可以在双引号里执行多条命令，命令之间用分号隔开；注意执行多条命令时必须用引号括起来；
35. pdsh -g mpp-jia tar xzf /tmp/mpp-patch.tgz -C `pwd` 以执行本命令的机器上的当前目录为目标目录（pwd命令的结果），在几个节点上解压tgz文件
36. tar -cvf xxx.tar.gz /dir #打包dir目录
37. tar -xvf xxx.tar.gz #解压缩
38. bunzip2 linux-2-4-2.tar.bz2 #解压生成 .tar文件
39. tar xvf linux-2-4-2.tar #解压 .tar文件
40. tar jxvf linux-2-4-2.tar.bz2 #一次性解压，等价于以上两条命令
41. cp -r /source /dest/ #将source目录复制到dest中，会在dest中自动新建一个source目录
42. cp -r /source/ /dest/ #将source中的子目录复制到dest中
43. env | grep hbase #查看hbase相关的环境变量配置信息
44. . xx.sh 和 ./xx.sh 优先使用前者,因为后者会新开一个子shell，在子shell里执行，而前者是在当前shell里执行
45. xx.sh == source xx.sh
46. ./xx.sh == sh xx.sh == bash xx.sh
47. rpm -qlp xxx.rpm #查看该rpm文件中包含的文件
48. rpm -qa| grep xxx #查看已经安装的rpm中名字包含xxx的
49. rpm -e xxx ( –nodeps ) #(强制)卸载，使用时一定要确认该rpm不被其他重要的软件所依赖，否则会导致系统故障！！！
50. rpm -e xxx #以普通用户身份执行，会因为权限不足/依赖关系而无法卸载，可以查看到依赖于xxx的程序
51. ssh user@hostname cmd #在远程主机上执行cmd命令，但不登录，直接返回结果，可能会遇到找不到命令的问题：  
	#[mpp2@metadb1 ~]$ ssh node10 jps  
	#bash: jps: command not found  
这是因为这种执行命令的方式和登陆到该主机上执行的方式所读取的配置文件不一样，所以环境变量有可能不同，这种方式会读取~/.bashrc 文件，所以需要在这里配置所需环境变量。详见：http://www.kuqin.com/shuoit/20141113/343188.html
52. shell 通配符：  
	ll seald.[EFIW][A-Z]* 中括号里面匹配任何一个字符，表示0个或多个，？表示一个
 	rm -f xxx.20151026-1[!5]\ 删除除了xxx-15*以外的其他文件
53. 在指定文件类型中查找关键字xxx：
	find . -name “.h” |xargs grep xxx
	注意.h要加双引号，否则不好使