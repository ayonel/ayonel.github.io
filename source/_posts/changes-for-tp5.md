---
title: 对thinkphp5.0 rc4的一些改动
url: 81.html
id: 81
categories:
  - PHP
date: 2016-08-19 18:27:40
tags:
  - PHP
---

&#160; &#160; &#160; &#160;最近开发一直用的是thinkphp5.0 rc4，个人感觉相较于tp3.x, tp5.0的进步是巨大的。
&#160; &#160; &#160; &#160;开发使用框架时，有时候需要根据自己的业务逻辑来框架源码做一些小改动，本文记录我在开发过程中对tp5.0 rc4做出修改的地方，如果更新框架，就需要对这些地方重新修改，特此MARK! 

***

> 1. Db类中使用分页输出数据的page()方法。

**改动缘由：**跟前台程序对接的时候，需要根据前台传入的$page变量来选择输出.例如page($page,10)，代表按10条数据分页，取第$page页。 
在开发的时候我发现$page传入0或者1，生成的sql都是一样的，取到的数据当然也一样。可能tp作者觉得默认开始的第一页就应该以1来表示，没有第0页。但跟前台约定的是:$page默认是0，所以$page=0应该是第一页，$pgae=1代表第二页。如果不做修改的话，$page第一次传0，下一次传1会返回相同的数据。
**改动位置：**/thinkphp/library/think/db/Query.php(2122-2129)：
```
if (isset($options['page'])) {
     // 根据页数计算limit
     list($page, $listRows) = $options['page'];
     $page                  = $page > 0 ? $page : 1;//对于传入的0它会自动变为1,需要改动
     $listRows              = $listRows > 0 ? $listRows : (is_numeric($options['limit']) ? $options['limit'] : 20);
     $offset                = $listRows * ($page - 1);
     $options['limit']      = $offset . ',' . $listRows;
}
```
**更改后：**
```
if (isset($options['page'])) {
     // 根据页数计算limit
     list($page, $listRows) = $options['page'];
     $page                  = $page >= 0 ? $page : 0;//传入的0会保持原来的0，并且其他非自然数类型也要自动变为0
     $listRows              = $listRows > 0 ? $listRows : (is_numeric($options['limit']) ? $options['limit'] : 20);
     $offset                = $listRows * $page;//$page-1改为$page
     $options['limit']      = $offset . ',' . $listRows;
}
``` 
> 2. 极光推送的默认日志记录文件

tp5在应用根目录下composer安装极光推送后，服务器一直500。经察是由于极光推送的日志文件jpush.log由于权限不够无法创建导致。手动建立该文件，并赋予766权限后还是报同样的错误。后来发现是由于极光推送的默认日志记录文件为【JPush.php中第16行】:
`const DEFAULT_LOG_FILE = "./jpush.log";` 
这个目录应该建立在tp5的入口文件的同级目录下。出于安全起见，我将tp5的入口文件部署在web目录下，而tp5核心框架及代码部署在非web目录下，所以我刚开始建立的jpush.log的位置错了。 可以手动指定绝对路径：
`const DEFAULT_LOG_FILE = "absolute_path";`