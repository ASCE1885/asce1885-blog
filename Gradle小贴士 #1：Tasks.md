# Gradle小贴士 #1：Tasks

> @author ASCE1885的 [Github](https://github.com/ASCE1885)  [简书](http://www.jianshu.com/users/4ef984470da8/latest_articles) [微博](http://weibo.com/asce885/profile?rightmod=1&wvr=6&mod=personinfo) [CSDN](http://blog.csdn.net/asce1885)
[@原文链接](http://trickyandroid.com/gradle-tip-1-tasks/)

在这篇文章中我将开始与Gradle相关的系列文章，希望有助于Gradle构建脚本的入门者。

今天将介绍Gradle的tasks，主要介绍task相关的配置和执行部分。由于绝大部分读者对这些概念不甚了解，因此我们从几个简单的例子入手。首先，看看下面三个例子有何不同：

```
task myTask {
    println "Hello, World!"
}

task myTask {
    doLast {
        println "Hello, World!"
    }
}

task myTask << {
    println "Hello, World!"
}
```

我的需求是创建一个执行时可以打印出"Hello, World!"的task。当第一次写脚本时，我的做法如下：

```
task myTask {
    println "Hello, World!"
}
```

现在，让我们来执行它！

```
user$ gradle myTask
Hello, World!
:myTask UP-TO-DATE
```

看起来脚本起作用了，它打印了"Hello, World!"。但实际上它并没有按预期的执行。为什么呢？让我们试着调用gradle tasks，来看看有没有其他可用的tasks：

```
user$ gradle tasks
Hello, World!
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
..........
```

等一下！为什么打印出了"Hello, World!"？我们刚才调用的是tasks，没有调用我们自定义的myTask啊！

原因在于，Gradle task的生命周期有两个主要的阶段：

* 配置阶段
* 执行阶段

这里所用的术语可能不是很准确，但这个类比可以帮助我们理解tasks。

Gradle脚本执行之前，需要首先配置构建脚本中指定的所有tasks。对于那些不需要执行的tasks，它依然需要被配置。既然如此，那么我的task中哪一部分会在配置阶段被配置，哪一部分会在执行阶段被执行呢？

答案是：在task的顶层声明的部分会在配置阶段进行配置，例如：

```
task myTask {
    def name = "Pavel" //<-- 在配置阶段配置
    println "Hello, World!"////<-- 也在配置阶段配置
}
```

这就是为什么当我们调用gradle tasks时，会打印出"Hello, World!"，这是myTask的配置阶段。显然，这并不是我们想要的，我想要myTask被真正执行时才打印出"Hello, World!"。

那么如何告诉Gradle在myTask被执行时才做某些事情呢？实现这个需求要指定task动作。指定task动作最简单的方式是通过Task的doLast()方法：

```
task myTask {
    def text = 'Hello, World!' //配置阶段
    doLast {
        println text //执行阶段
    }
}
```

现在只有在我们显式调用gradle myTask时才会打印出"Hello, World!"。赞，我们知道如何配置以及使myTask在我们触发它时才执行实际的工作。那使用<<符号的第三个例子是啥意思呢？

```
task myTask2 << {
    println "Hello, World!" 
}
```

这个版本只是doLast版本的简化版，也就是说，它跟下面这段脚本功能是完全一样的：

```
task myTask {
    doLast {
        println 'Hello, World!' //this is executed when my task is called
    }
}
```

然而，第三个版本所有的代码都在执行阶段进行，我们不能像doLast版本那样在配置阶段做一些定义（事实上，仍然可以实现de，但是以一种稍微不同的方式），因此这个版本适合不需要配置的小任务，当你的task比单纯的打印"Hello, World!"复制时，你可能更应该使用doLast版本。

享受Gradle编程吧！




