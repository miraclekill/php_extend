# php_extend
win7 vs2013 c开发php-5.6.38-nts-Win32-VC11-x86扩展       
# vs2013 对应的是VC12   
1 下载php-5.6.38-src源码, php-5.6.38-nts-Win32-VC11-x86编译好的二进制包文件,需要注意的是x86,因为是win32,不能下载成x64,   
后面需要引用php5.lib,如版本不对应会报错.下载地址https://windows.php.net/download/   
2 安装Cygwin（下载地址：http://www.cygwin.com/）  
3 安装vs2013  
4 修改ext_skel_win32.php中$cygwin_path=程序所安装路径  
5 生成骨架  cd php-5.6.38-src\ext;    进入路径后执行 php-5.6.38-nts-Win32-VC11-x86\php.exe ext_skel_win32.php --extname=test,test是扩展名称  
6 vs 新建->从现有代码文件创建项目->c++->生成的扩展路径->动态链接dll->完成  
7 修改错误,  
1)先把项目解决方案配置改为Release,右键项目属性，C/C++，常规，附加包含目录，编辑.   
2)加入以下几个php源码目录  
php-5.6.38-src,  
php-5.6.38-src\main,  
php-5.6.38-src\zend,  
php-5.6.38-src\TSRM  
3)右键项目属性，C/C++，预处理器，预处理器定义，编辑，加入以下变量：  
ZEND_DEBUG=0  
PHP_EXTENSION  
PHP_WIN32  
ZEND_WIN32  
HAVE_TEST=1（这里红色部分，要改成你的扩展名称，不改成你的扩展名，php会不识别）  
COMPILE_DL_TEST（这里红色部分，要改成你的扩展名称，不改成你的扩展名，php会不识别）  
ZTS（这一个变量加上是开启线程安全，不加是关闭线程安全）nts 不用添加, ts需要添加   
8 拷贝win32\build\config.w32.h.in => main\config.w32.h  
9 右键项目属性，连接器，输入，附加依赖项，编辑，将php5.lib的路径放进去,此处需要注意因为是win32所有必须是x86下的也就是  
php-5.6.38-nts-Win32-VC11-x86下的dev\php5.lib,不然报错  
10 具体配置见 https://blog.csdn.net/muyilongh/article/details/51062262  
11 编写代码  
PHP_FUNCTION(miracle_add) {  
  int argc = ZEND_NUM_ARGS();
	int x;
	int y;
	int z;
	if (zend_parse_parameters(argc TSRMLS_CC, "ll", &x, &y) == FAILURE) {   //ll此处是L的小写
		return;
	}
	z = x + y;
	RETURN_LONG(z);
}
PHP_FUNCTION(miracle_concat)
{
	char *str = NULL;
	int argc = ZEND_NUM_ARGS();
	int str_len;
	long n;
	char *result;
	char *ptr;
	int result_length;

	if (zend_parse_parameters(argc TSRMLS_CC, "sl", &str, &str_len, &n) == FAILURE) {
		return;
	}

	result_length = (str_len * n);
	result = (char *)emalloc(result_length + 1);
	ptr = result;
	while (n--) {
		memcpy(ptr, str, str_len);
		ptr += str_len;
	}

	*ptr = '\0';

	RETURN_STRINGL(result, result_length, 0);
}

const zend_function_entry jia_functions[] = {
	PHP_FE(miracle_add, NULL)  //添加上面对应的方法名称且不能为最后一行
	PHP_FE(miracle_concat, NULL) //添加上面对应的方法名称且不能为最后一行
	PHP_FE(confirm_jia_compiled,	NULL)		/* For testing, remove later. */
	PHP_FE_END	/* Must be the last line in jia_functions[] */
};
12 生成xx.dll文件  
13 把xx.dll文件拷贝运行环境ext下 配置php.ini extension=xx.dll   
14 重启服务  
15 php调用   
<?php 
  echo miracle_add(4, 19);
  echo miracle_concat("abcd", 20);
