#背景
今天QA说开发测试用的数据库查不到数据了，登录看了一下，普通查询正常，但是执行特定的一个语句就会报错，错误信息：
```
Incorrect key file for table '/tmp/#sql_5608_1.MYI'; try to repair it
```
根据关键词百度了一下，尝试使用repair命令修复，命令：
```
mysql> repair table test USE_FRM;
```
由于表的存储引擎都是innodb，所以不好使，将引擎改为myisam之后，在mysqld服务机器执行以下命令，
```
myisamchk -of /var/lib/mysql/test/test.MYI
myisamchk -r /var/lib/mysql/test/test.MYI
myisamchk safe-recover /var/lib/mysql/test/test.MYI
```
依然无效，可能对于myisam引擎，该错误可以用上述方法解决，但我的问题没解决

参考文档
http://www.cnblogs.com/zjoch/archive/2013/08/19/3267131.html

#排查
参考文档：
http://blog.itpub.net/26355921/viewspace-1273862/

根据文档中的描述，查看了/tmp空间只剩15M，而我的报错的语句执行了一个500k*500k的联表查询，因此猜测可能是因为tmp空间不足导致报错，
根据参考文档中的提示，执行查询期间，df看了下/tmp目录的占用，果然飙升到100%，那么问题定位，就是tmpdir目录引起的。

#修复
在my.cnf中增加以下配置：
```
tmpdir = /home/work/mysql/tmp/
```
将tmpdir转移到home分区下，重启mysqld服务后，查询语句正常

#总结
在linux系统下，尽量避免使用/tmp目录当做存储使用，一般/tmp目录下只用来存储临时文件，要及时清理
