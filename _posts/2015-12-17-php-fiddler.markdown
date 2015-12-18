---
layout:    post
title:     "Fiddler分拆大型代码库"
subtitle:  "拆分出各种粒度的composer包来组件化和依赖管理"
date:       2015-12-17 10:16
category: php
---
> 根据`宝峰-Composer在一淘社区中的应用` 的内容进行实践
> [Fiddler.phar下载地址](https://github.com/beberlei/fiddler/releases)

### 下载安装

下载fiddler.phar 后移动该文件到/usr/local/bin/目录下 `mv fiddler.phar /usr/local/bin/fiddler` `chmod +x /usr/local/bin/fiddler`
此时就可以直接执行fiddler命令， 否则只能执行 `php fiddler.phar`

### 实例

目录结构：

```
└──project
   ├── composer.json
   ├── composer.lock
   ├── projects
   │   └── components        // 组件包目录
   │       ├── Library_1     // 公共类库1
   │       │   └── fiddler.json
   │       ├── Project_A        // 项目A组件
   │       │   └── fiddler.json
   │       └── Project_B        // 项目B组件
   │           └── fiddler.json
   └── vendor   // vendor上的目录比较多这里就不列出来了 
```

Library_1/fiddler.json内容

{% highlight javascript startinline %} 
{
    "autoload": {
        "psr-0": {  // fig组织定义的psr-0自动加载规范
            "Libra1\\": "src/"  
        }
    },
    "deps": [ // 依赖组件列表
        "vendor/symfony/dependency-injection" // composer生成的vendier目录下的组件巨鹿路径
    ]
}
{% endhighlight %}
      
Project_A/fiddler.json内容

{% highlight javascript startinline %} 
 {
      "autoload" : {
            "psr-0": {  // fig组织定义的psr-0自动加载规范
                "ProjectA\\":"src/" 
            }
      },
      "deps": [ // 依赖组件列表
            "components/Library_1"  // 依赖Library_1类库， 即Library_1自己定义的自动加载， Project_A都能使用
      ]
 }
{% endhighlight %}

### 执行fiddler脚本
filder脚本执行一定要在composer.json同级的目录下执行, 即 本`project/vendor/`

执行过程：

```
cp@cp:/var/www/fiddler/projects$ sudo fiddler build
Building fiddler.json projects.
 [Build] components/Library_1
 [Build] components/Project_B
 [Build] components/Project_A
```

执行结果：

```     
 cp@cp:/var/www/fiddler/projects/components$ tree
 .
 ├── Library_1
 │   ├── fiddler.json
 │   └── vendor
 │       ├── autoload.php
 │       └── composer
 │           ├── autoload_classmap.php
 │           ├── autoload_namespaces.php
 │           ├── autoload_psr4.php
 │           ├── autoload_real.php
 │           └── ClassLoader.php
 ├── Project_A
 │   ├── fiddler.json
 │   └── vendor
 │       ├── autoload.php
 │       └── composer
 │           ├── autoload_classmap.php
 │           ├── autoload_namespaces.php
 │           ├── autoload_psr4.php
 │           ├── autoload_real.php
 │           └── ClassLoader.php
 └── Project_B
     ├── fiddler.json
     └── vendor
         ├── autoload.php
         └── composer
             ├── autoload_classmap.php
             ├── autoload_namespaces.php
             ├── autoload_psr4.php
             ├── autoload_real.php
             └── ClassLoader.php
 ```

执行完后仅生成各自的自动加载文件， 所有的依赖包都的 `project/vendor` 目录下