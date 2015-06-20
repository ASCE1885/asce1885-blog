# Android Studio分模块自动化构建实战

> @author ASCE1885的 [Github](https://github.com/ASCE1885)  [简书](http://www.jianshu.com/users/4ef984470da8/latest_articles) [微博](http://weibo.com/asce885/profile?rightmod=1&wvr=6&mod=personinfo) [CSDN](http://blog.csdn.net/asce1885)

最近在使用Android Studio＋Gradle做一个基础框架SDK项目，该框架主要实现每个app都需要的基础能力，例如网络请求，图片缓存，json解析，日志记录等等。

众所周知，AndroidStudio中应该尽量使用Module来进行模块的划分，既能达到模块解耦的目的，也能在必要的时候轻松实现分模块打包，特别是在SDK项目中。那么什么是分模块打包呢？就是我们可以根据第三方使用者的需求，自动化的提供SDK的全量版本，部分功能版本以及最小功能版本等等。

我们的项目结构如下所示，每个功能独立成一个Module：

![](http://img.blog.csdn.net/20150619173727389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

由于我们的模块都是纯代码的，没有包含资源文件，因此不是以aar包的形式而是使用jar包形式对外提供。顺便提一句，生成的aar包默认路径是：

```
build/output/aar/
```

而jar包可以到如下路径寻找：

```
build/intermediates/bundles/debug/classes.jar
build/intermediates/bundles/release/classes.jar
```

###Jar包的合并

从项目工程截图中可以看到，我们的project包含多个module，每个基础功能的module最终编译生成的都是一个classes.jar。因此project最终会生成一堆的jar包，而到了对外发布时，我们要提供一个单独的jar包出去，因此，就需要对jar包进行合并。很不幸，Android Studio没有提供这样的功能，因此只能靠自己写脚本调用jar命令来实现了，打开命令行terminal，输入jar，就可以打印出jar的用法，如下所示：

```
guhaoxindeMacBook-Pro:~ guhaoxin$ jar
用法: jar {ctxui}[vfm0Me] [jar-file] [manifest-file] [entry-point] [-C dir] files ...
选项包括: 
    -c  创建新的归档文件
    -t  列出归档目录
    -x  从档案中提取指定的 (或所有) 文件
    -u  更新现有的归档文件
    -v  在标准输出中生成详细输出
    -f  指定归档文件名
    -m  包含指定清单文件中的清单信息
    -e  为捆绑到可执行 jar 文件的独立应用程序
        指定应用程序入口点
    -0  仅存储; 不使用情况任何 ZIP 压缩
    -M  不创建条目的清单文件
    -i  为指定的 jar 文件生成索引信息
    -C  更改为指定的目录并包含其中的文件
如果有任何目录文件, 则对其进行递归处理。
清单文件名, 归档文件名和入口点名称的指定顺序
与 'm', 'f' 和 'e' 标记的指定顺序相同。

示例 1: 将两个类文件归档到一个名为 classes.jar 的归档文件中: 
       jar cvf classes.jar Foo.class Bar.class 
示例 2: 使用现有的清单文件 'mymanifest' 并
           将 foo/ 目录中的所有文件归档到 'classes.jar' 中: 
       jar cvfm classes.jar mymanifest -C foo/ .
```

使用jar命令，主要实现两个功能：

* 将所有jar包的class文件解压到某个目录中
* 将解压后所有class文件的重新压缩为一个单独的jar包

由于jar命令不能指定最终输出的目录，因此我们需要首先cd到用于存放解压后class文件的一个临时目录，然后依次对所有jar包进行解压操作，解压命令如下所示：

```
jar -xvf ../../hfasynchttp/build/intermediates/bundles/debug/classes.jar
```

当所有的jar包都解压完毕后，接着执行压缩命令，这样就得到一个单独的jar包了：

```
jar -cvfM AndroidHyperion_${version}_debug.jar .
```

###分模块自动化构建

自动化构建包括本地构建和Jenkins构建两部分，本地构建主要用于开发自己调试使用，Jenkins构建主要用于测试，产品等取包以及跑Monkey使用。

####本地构建

本地构建脚本文件位于工程根目录下的build_local.sh，该脚本的主要功能有：

* 调用gradlew命令执行gradle编译，生成各个Module的jar包
* 解压各个Module生成的jar包，并把解压后的class文件重新打包成单独的一个jar包
* 分模块打包功能通过定义Boolean变量值进行控制
* 输出目录是output

build_local.sh文件内容如下：

```
#!/bin/sh

#使用Gradle编译各个module
./gradlew clean
./gradlew build --stacktrace --debug

#进入输出目录
cd output

#清空输出目录
rm -rf  *

#创建输出子目录
mkdir temp
mkdir debug
mkdir release

#定义sdk版本号
version="1.0.0"

#定义模块是否打包标识
is_include_hfasynchttp=true
is_include_bitmapfun=true
is_include_hfjson=true
is_include_hflogger=true
#省略其他...

#解压所有debug版本的jar包到temp目录中
cd temp

if $is_include_hfasynchttp; then
    jar -xvf ../../hfasynchttp/build/intermediates/bundles/debug/classes.jar
fi

if $is_include_bitmapfun; then
    jar -xvf ../../hfbitmapfun/build/intermediates/bundles/debug/classes.jar
fi

if $is_include_hfjson; then
    jar -xvf ../../hfjson/build/intermediates/bundles/debug/classes.jar
fi

if $is_include_hflogger; then
    jar -xvf ../../hflogger/build/intermediates/bundles/debug/classes.jar
fi

#压缩所有debug版本的class文件到一个独立的jar包中
jar -cvfM AndroidHyperion_${version}_debug.jar .

#拷贝文件
mv AndroidHyperion_${version}_debug.jar ../debug

#清空temp目录
rm -rf *

#解压所有release版本的jar包到temp目录中
if $is_include_hfasynchttp; then
    jar -xvf ../../hfasynchttp/build/intermediates/bundles/release/classes.jar
fi

if $is_include_bitmapfun; then
    jar -xvf ../../hfbitmapfun/build/intermediates/bundles/release/classes.jar
fi

if $is_include_hfjson; then
    jar -xvf ../../hfjson/build/intermediates/bundles/release/classes.jar
fi

if $is_include_hflogger; then
    jar -xvf ../../hflogger/build/intermediates/bundles/release/classes.jar
fi

#压缩所有release版本的class文件到一个jar包中
jar -cvfM AndroidHyperion_${version}_release.jar .

#拷贝文件
mv AndroidHyperion_${version}_release.jar ../release

#删除temp目录
cd ..
rm -rf temp
```

####Jenkins构建

Jenkins编译脚本文件位于工程根目录下的build_jenkins.sh，该脚本的主要功能有：

* 调用gradlew命令执行gradle编译，生成各个Moudle的jar包
* 解压各个Module生成的jar包，并把解压后的class文件重新打包成单独的一个jar包
* 分模块打包功能通过Jenkins上面配置的参数化构建参数进行控制
* 输出目录是output

可以看到，和本地构建唯一的区别就是分模块的参数化构建参数是定义在Jenkins上的，而不是定义在本地脚本中的，为了完整清晰起见，我们还是把完整的脚本文件贴出来：

```
#!/bin/sh

./gradlew clean
./gradlew build --stacktrace --debug

#进入输出目录
cd output

#清空输出目录
rm -rf  *

#创建输出子目录
mkdir temp
mkdir debug
mkdir release

cd temp

#解压所有debug版本的jar包
if $is_include_hfasynchttp; then
    jar -xvf ../../hfasynchttp/build/intermediates/bundles/debug/classes.jar
fi

if $is_include_bitmapfun; then
    jar -xvf ../../hfbitmapfun/build/intermediates/bundles/debug/classes.jar
fi

if $is_include_hfjson; then
    jar -xvf ../../hfjson/build/intermediates/bundles/debug/classes.jar
fi

if $is_include_hflogger; then
    jar -xvf ../../hflogger/build/intermediates/bundles/debug/classes.jar
fi

#压缩所有debug版本的class文件到一个jar包中
jar -cvfM AndroidHyperion_${version}_debug.jar .

#移动生成的jar包到debug目录
mv AndroidHyperion_${version}_debug.jar ../debug

#清空temp目录
rm -rf *

#解压所有release版本的jar包
if $is_include_hfasynchttp; then
    jar -xvf ../../hfasynchttp/build/intermediates/bundles/release/classes.jar
fi

if $is_include_bitmapfun; then
    jar -xvf ../../hfbitmapfun/build/intermediates/bundles/release/classes.jar
fi

if $is_include_hfjson; then
    jar -xvf ../../hfjson/build/intermediates/bundles/release/classes.jar
fi

if $is_include_hflogger; then
    jar -xvf ../../hflogger/build/intermediates/bundles/release/classes.jar
fi

#压缩所有release版本的class文件到一个jar包中
jar -cvfM AndroidHyperion_${version}_release.jar .

#移动生成的jar包到release目录
mv AndroidHyperion_${version}_release.jar ../release

#删除temp目录
cd ..
rm -rf temp

```

####local和Jenkins参数化构建参数定义

| 类型       | 名称                           |  默认值    | 描述 
| --------   | -----:                          | :----:     | :----:  
| String     | version                         |   1.0.0    | Hyperion sdk版本号 
| Boolean    | is_include_hfasynchttp          | true       | 是否打包hfasynchttp 
| Boolean    | is_include_bitmapfun            | true       | 是否打包hfbitmapfun 
| Boolean    | is_include_hfjson               | true       | 是否打包hfjson 
| Boolean    | is_include_hflogger             | true       | 是否打包hflogger
