#简介
xhprof是facebook开发的一款函数级别的php性能分析工具，可以看到每个函数的调用次数和执行时间，方便优化php性能

#安装
项目地址：http://pecl.php.net/package/xhprof
```shell
wget http://pecl.php.net/get/xhprof-0.9.2.tgz
tar zxf xhprof-0.9.2.tgz
cd xhprof-0.9.2/extension/
sudo $PHPIZE-PATH  #这里是phpize的路径，根据php的安装路径来找，一般是跟php文件在一起的，如/usr/local/bin/phpize之类的
./configure --with-php-config=$PHP-CONFIG-PATH  #这里是php-config的路径，跟phpize在一起，如/usr/local/bin/php-config之类的
make
make install  #这一步默认是安装在/usr/local目录下面，如果没有root权限，则在configure里面需要指定一下prefix，如：--prefix=/home/work
```
安装后，会将so文件直接复制到php的扩展库下面，文件为：xhprof.so

#配置php
在php.ini文件中添加xhprof.so扩展：
```
extension=xhprof.so;
```
#配置测试文件
在要测试的php文件中加入以下代码，如test.php：

```php
<?php
// cpu:XHPROF_FLAGS_CPU 内存:XHPROF_FLAGS_MEMORY
// 如果两个一起：XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY 
xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);

// 要测试的php代码

// 测试代码结束

$data = xhprof_disable();   //返回运行数据
 
// xhprof_lib在下载的包里存在这个目录,记得将目录包含到运行的php代码中
include_once "xhprof_lib/utils/xhprof_lib.php";  
include_once "xhprof_lib/utils/xhprof_runs.php";  
 
$objXhprofRun = new XHProfRuns_Default(); 

// 第一个参数j是xhprof_disable()函数返回的运行信息
// 第二个参数是自定义的命名空间字符串(任意字符串),
// 返回运行ID,用这个ID查看相关的运行结果
$run_id = $objXhprofRun->save_run($data, "xhprof");
var_dump($run_id);
```

#查看测试结果
将xhprof_lib&&xhprof_html相关目录copy到可以访问到的地址

访问 http://xxx/xhprof_html/index.php?run=$run_id&source=xhprof 

就可经看到你的php代码运行的相关情况，注意source参数与save_run的第二个参数一致

下面是一些参数说明
```
Inclusive Time                  包括子函数所有执行时间。
Exclusive Time/Self Time        函数执行本身花费的时间，不包括子树执行时间。
Wall Time                       花去了的时间或挂钟时间。
CPU Time                        用户耗的时间+内核耗的时间
Inclusive CPU                   包括子函数一起所占用的CPU
Exclusive CPU                  函数自身所占用的CPU
```
