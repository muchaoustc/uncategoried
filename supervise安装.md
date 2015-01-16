# supervise的简介请百度

以下为安装步骤:

```shell
mkdir /package 
chmod 1755 /package 
cd /package 
wget http://cr.yp.to/daemontools/daemontools-0.76.tar.gz 
tar zxf daemontools-0.76.tar.gz 
rm -f daemontools-0.76.tar.gz 
cd admin/daemontools-0.76 
package/install 
```

如果你的glibc库比较新，此时你很可能会遇到下面错误： 
```shell
/usr/bin/ld: errno: 
TLS definition in /lib/libc.so.6 section .tbss mismatches non-TLS reference in envdir.o 
/lib/libc.so.6: could not read symbols: Bad value 
collect2: ld returned 1 exit status 
make: *** [envdir] Error 1 
Copying commands into ./command... 
cp: cannot stat `compile/svscan': No such file or directory 
```

为了解决这个问题，接着上面的安装步骤继续：
```shell
cd src 
wget http://www.qmail.org/moni.csi.hu/pub/glibc-2.3.1/daemontools-0.76.errno.patch 
patch < daemontools-0.76.errno.patch 
cd .. 
package/install 
```
也可以采取手动修改的方式： 

```shell
vi src/conf-cc
``` 
在最后加上 -include /usr/include/errno.h 
缺省会安装到/usr/local/bin目录，最好加到PATH环境变量里。 

可以选择安装man手册：

```shell
wget http://smarden.org/pape/djb/manpages/daemontools-0.76-man.tar.gz 
tar zxf daemontools-0.76-man.tar.gz 
cd daemontools-man 
gzip *.8 
cp *.8.gz /usr/share/man/man8 
```

运行方式是，在程序目录下手动创建supervise的目录结构，比如
/home/work/hhvm-3.0.1/ # the service directory 
/home/work/hhvm-3.0.1/run # the script that starts lighttpd 
/home/work/hhvm-3.0.1/log/ # the service directory for the logger, optional
/home/work/hhvm-3.0.1/log/run # the script that starts the logger, optional
/home/work/hhvm-3.0.1/log/main/ # log files go here, optional

run是supervise执行的唯一入口, 可以简单写如下脚本：
```shell
#! /bin/sh
exec 2>&1
exec /home/work/hhvm-3.0.1/bin/hhvm --mode server \
--config /home/work/hhvm-3.0.1/conf/hhvm.conf \
--config-value PidFile=/home/work/hhvm-3.0.1/var/hhvm.pid
```
可以根据自己的程序启动命令修改，run脚本需要有执行权限：
```shell
chmod +x /home/work/hhvm-3.0.1/run
```
启动：
```shell
nohup supervise  /home/work/hhvm-3.0.1/ &
```
因为supervise是独占运行的，因此需要加上nohup后台执行
