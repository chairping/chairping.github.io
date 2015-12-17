---
layout:    post
title:     "Composer"
subtitle:  "PHP 用来管理依赖（dependency）关系的工具。"
date:       2015-12-17 10:16
category: php
---

> [composer中文文档](http://docs.phpcomposer.com/00-intro.html)


### 简介
Composer是PHP的一个依赖管理工具。它允许你申明项目所依赖的代码库，它会在你的项目中为你安装他们。 有Composer后，你就可以直接在[https://packagist.org/](https://packagist.org/)上面找符合自己项目的开源包（不用重复造轮子），然后通过composer去下载安装该依赖包，下载即可使用， 因为composer已经帮你做好了自动加载。


### 下载安装

下载compsoer `curl -sS https://getcomposer.org/installer | php`  
直接移动下载下来的composer.phar到bin目录下，`mv composer.phar /usr/local/bin/composer` 这样我们可直接执行compoer命令

### 使用

* 根目录初始化(在根目录www下创建一个`composer.json`) 

         [root@centos www]# ls
         composer.json 
             
    composer.json 文件里面的内容：

        {
            "require": {
                "monolog/monolog": "1.2.*"
            }
        }

* 在www目录下执行命令 `composer install`
    执行过程:    
      
        [root@centos www]# composer install 
        Installing dependencies (including require-dev)
          - Installing monolog/monolog (1.2.1)
            Downloading: 100%         
            Downloading: 100%         
        
        monolog/monolog suggests installing mlehner/gelf-php (Allow sending log messages to a GrayLog2 server)
        monolog/monolog suggests installing ext-amqp (Allow sending log messages to an AMQP server (1.0+ required))
        Writing lock file
        Generating autoload files
    
    执行结果:    
    根据执行过程， `composer install` 命令会自动下载`monolog/monglog`包，  `Writing lock file`表示生成一个composer.lock 锁包，
    `Generating autoload files` 表示生成一个自动加载文件。
    
     当前目录结构:
     
        [root@centos www]# tree
        ├── composer.json
        ├── composer.lock
        └── vendor
            ├── autoload.php  // 类自动加载入口文件
            ├── composer      // 类自动加载相关文件
            │   ├── autoload_classmap.php    
            │   ├── autoload_namespaces.php  
            │   ├── autoload_psr4.php
            │   ├── autoload_real.php
            │   ├── ClassLoader.php      
            │   ├── installed.json
            │   └── LICENSE
            └── monolog      // monolog 依赖包的根目录 
                └── monolog
                    ├── CHANGELOG.mdown
                    ├── composer.json
                    ├── doc
                    ├── LICENSE
                    ├── phpunit.xml.dist
                    ├── README.mdown
                    └── src

 * 自动加载
    你只需要将下面这行代码添加到你项目的引导文件中即可实现自动加载：
    
        require 'vendor/autoload.php';

 * 常用命令 [点击查看完整命令详情](http://docs.phpcomposer.com/03-cli.html)
   更新全部依赖包`composer update` 更新指定依赖包`composer update monolog/monolog` 
   不修改compser包直接下载安装`composer.phar require vendor/package:2.*`
   搜索依赖包`composer search` 
   composer自己更新版本 `composer self-update`

     
         