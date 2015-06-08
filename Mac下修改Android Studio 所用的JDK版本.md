# Mac下修改Android Studio 所用的JDK版本

> @author [ASCE1885](https://github.com/ASCE1885)

最近项目从Eclipse＋Ant构建模式转移到了Android Studio＋Gradle构建模式，自然的JDK版本号也从JDK6升级到了JDK7，但后来发现，由于我们是一个SDK项目，最终会以JAR包形式提供给第三方使用，这样就会遇到一个问题，如果我们使用JDK7编译JAR包，而第三方编译环境使用的还是旧的JDK6，那么编译工程的时候就会出现：

```
Unsupported major.minor version 51.0
```

因此需要把我们的Android Studio工程编译环境从JDK7降为JDK6，下面就是修改记录。

###Mac系统JDK不同版本的路径

默认情况下，Mac系统JDK不同版本的默认安装目录有点差别，JDK6，JDK7和JDK8的 安装目录分别如下所示：

```
/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk1.7.0.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk1.8.0.jdk/Contents/Home
```

###Android Studio的修改

点击Android Studio的File-Other Settings-Default Project Structure：

![](http://img.blog.csdn.net/20150604171658327?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

打开Project Structure对话框，在这个对话框中可以修改Android SDK和JDK的路径：

![](http://img.blog.csdn.net/20150604171945411?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

###环境变量的修改

经过上面的修改，我们在Android Studio UI界面上编译时，将使用我们修改后的JDK6版本，但是当我们在Terminal中输入java -version查看当前JDK版本信息时，会发现还是之前的JDK7版本，可能的原因是之前系统中设置了JAVA_HOME环境变量，因此这里也要修改一下。

![](http://img.blog.csdn.net/20150604173440845?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

打开Terminal，输入vim ~/.bash_profile，打开这个文件，内容如下：

![](http://img.blog.csdn.net/20150604173713944?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到，环境变量里面确实还是之前的JDK7版本，将其指向JDK6的安装路径就可以了。然后重新加载profile使其生效：

```
source ~/.bash_profile
```

###Jenkins的修改

自动化编译使用的是Tomcat＋Jenkins，Jenkins的修改比较简单，在Jenkins首页点击[系统管理]－[系统设置]，找到如下JDK设置选项进行修改即可：

![](http://img.blog.csdn.net/20150605163511881?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

修改完成之后，重启Tomcat，会发现Jenkins页面访问失败，原因在于我们使用的Jenkins版本最低只支持JRE7，官网截图如下：

![](http://img.blog.csdn.net/20150605163753693?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXNjZTE4ODU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

而在环境变量那一步我们已经将JDK版本从JDK7降为JDK6了，这时可以通过修改Tomcat的JRE_HOME参数来解决，打开Tomcat安装目录下bin/catalina.sh（因为我使用的是Mac系统，Windows系统请切换到catalina.bat），搜索JRE_HOME参数，如果不存在就新建之，并赋值为电脑上JDK7目录：

```
JRE_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_75.jdk/Contents/Home
```




