---
layout:    post
title:     "php fileinfo 学习笔记"
subtitle:  "文件相关扩展fileinfo"
date:       2015-12-15 11:02
---


finfo 预定义常量
---------------------

`FILEINFO_NONE`         无特殊处理。
`FILEINFO_SYMLINK`      跟随符号链接。
`FILEINFO_MIME_TYPE`    返回 mime 类型。
`FILEINFO_MIME_ENCODING` 返回文件的 mime 编码。
`FILEINFO_MIME`         按照 RFC 2045 定义的格式返回文件 mime 类型和编码。
`FILEINFO_COMPRESS`     解压缩压缩文件。 由于线程安全问题，自 PHP 5.3.0 禁用。
`FILEINFO_DEVICES`      查看设备的块内容或字符。
`FILEINFO_CONTINUE`     返回全部匹配的类型。
`FILEINFO_PRESERVE_ATIME` 如果可以的话，尽可能保持原始的访问时间。
`FILEINFO_RAW`          对于不可打印字符不转换成 \ooo 八进制表示格式。

返回内容根据表格如下:

| 文件名                | 返回文件的mime编码 | mine类型       |文件mime类型和编码(RFC2045定义的格式)|FILEINFO_RAW&&全部匹配的类型&&无特殊处理&&跟随符号链接&&FILEINFO_DEVICES|
| -------------------- |:---------------:|:-------------:|--------------------------------|:----------------------------------------------------------:|
| avatar2.png          | binary          | image/png     | image/png; charset=binary      | PNG image data, 215 x 215, 8-bit colormap, non-interlaced  |
| bootstrap-slider.js  | us-ascii        | text/plain    | text/plain; charset=us-ascii   | ASCII text                                                 |
| boxed-bg.jpg         | binary          | image/jpeg    | image/jpeg; charset=binary     | PEG image data, JFIF standard 1.01                         |
| default-50x50.gif    | binary          | image/gif     | image/gif; charset=binary      | GIF image data, version 87a, 50 x 50                       |
| index.php            | us-ascii        | text/x-php    | text/x-php; charset=us-ascii   | PHP script, ASCII te                                       |
| slider.css           | us-ascii        | text/plain    | text/plain; charset=us-ascii   | ASCII text                                                 |
| test.html            | us-ascii        | text/html     | text/html; charset=us-ascii    | HTML document, ASCII text                                  |

读取文件资源类型 有两种方法
---------------------

函数简介: `finfo_open`    创建一个fineinfo资源
函数简介: `finfo_file`    返回文件信息  `finfo_close`   关闭finfo_open创建的文件资源
函数简介: `finfo_buffer`  返回一个字符串缓冲区的信息

*   方法一：

    ```php
    $finfo = finfo_open(FILEINFO_MIME_TYPE);        // 返回 mime 类型
    foreach (glob("fileinfo/*") as $filename) {
        echo finfo_file($finfo, $filename) . "<br />"; // 输出mine信息
    }
    finfo_close($finfo);

    // or 面向对象风格

    $finfo = new \finfo(FILEINFO_MIME_TYPE);
    foreach (glob("fileinfo/*") as $filename) {
        echo  $finfo->file($filename) . "<br />";
    }
    ```

*   方法二：

    ```php
    $string = file_get_contents('/www/cp/public/fileinfo/boxed-bg.jpg');// 读取文件返回字符串内容
    echo finfo_buffer(finfo_open(FILEINFO_MIME_TYPE), $string);         // 通过finfo_bufer 识别内容

    // or 面向对象风格

    $fileinfo = new \finfo(FILEINFO_MIME_TYPE);
    echo $fileinfo->buffer(file_get_contents('/www/cp/public/fileinfo/boxed-bg.jpg'));
    ```


http://www.jianshu.com/p/609e1197754c  jekyll创建博客





