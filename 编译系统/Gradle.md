和Maven一样，Gradle只是提供了构建项目的一个框架，真正起作用的是Plugin。Gradle在默认情况下为我们提供了许多常用的Plugin，其中包括有构建Java项目的Plugin，还有War，Ear等。与Maven不同的是，Gradle不提供内建的项目生命周期管理，只是java Plugin向Project中添加了许多Task，这些Task依次执行，为我们营造了一种如同Maven般项目构建周期rty以完成特定的operty以完成特定的操作。

现在我们都在谈领域驱动设计，Gradle本身的领域对象主要有Project和Task。Project为Task提供了执行上下文，所有的Plugin要么向Project中添加用于配置的Property，要么向Project中添加不同的Task。一个Task表示一个逻辑上较为独立的执行过程，比如编译Java源代码，拷贝文件，打包Jar文件，甚至可以是执行一个系统命令或者调用Ant。另外，一个Task可以读取和设置Project的Property以完成特定的操作。


让我们来看一个最简单的Task，创建一个build.gradle文件，内容如下：

task helloWorld << {
   println "Hello World!"
}

在与build.gradle相同的目录下执行：

gradle helloWorld

输出：

:helloWorld
Hello World!

BUILD SUCCESSFUL

Total time: 2.544 secs

Gradle在默认情况下为我们提供了几个常用的Task，比如查看Project的Properties、显示当前Project中定义的所有Task等。可以通过一下命令查看Project中所有的Task：

gradle tasks

