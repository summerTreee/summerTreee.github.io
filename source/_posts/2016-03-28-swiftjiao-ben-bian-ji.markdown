---
layout: post
title: "Swift脚本编辑"
date: 2016-03-28 07:23:47 +0800
comments: true
categories: 
---

`Swift`是一种编译型语言，功能强大而且灵活性十足，现在你甚至可以在`shell`脚本中运行`Swift`代码。  
  
编写`shell`脚本包括以下几个步骤：  
  
1. 使用编辑器创建脚本。
2. 设置脚本的权限，使其能够执行。
3. 执行脚本。  
<!--more-->
####1、编辑脚本  
我们可以使用`Xcode`来直接创建`shell`脚本：  
  
选择菜单 `File -> New -> File`,在`iOS`下选择`Other`，再在右边选择`Shell Script`    
  
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f2c7xkqp1vj20kc0ehq4t.jpg)  
  
单击`Next`按钮，将新建的文件保存，命名为**SwiftScript**，Xcode会自动为其添加`.sh`的后缀名。这时你就可以在新出现的窗口中编辑脚本了。  
  
`Xcode`将自动为这个文件添加几行代码，其中最重要的一行是：  
  
	#!/bin/sh  
  
这被称为`hash bang`语法，指定了要用来运行后续代码行的`shell`在文件系统中的完整路径。这里指定的是`/bin/sh（Bash shell）`。进行`Swift`脚本编程时，需要移除这行代码。事实上，请删除所有这些代码行，并输入如下代码行：  
  
```
#!/usr/bin/env xcrun swift
import Foundation

class Execution {
class func execute(path path: String, arguments: [String]? = nil) -> Int {
   let task = NSTask()
   task.launchPath = path
   if arguments != nil {
     task.arguments = arguments!
   }

   task.launch()
   task.waitUntilExit()
   return Int(task.terminationStatus)

}
}

var status : Int = 0
status = Execution.execute(path: "/bin/ls")
print("Status = \(status)")

status = Execution.execute(path: "/bin/ls", arguments:["/"])
print("Status = \(status)")

```  
  
该脚本的含义将在稍后介绍。  
  
####2、设置权限
脚本是从命令行运行的，所以启动`Terminal`，进入到你刚才所写**shell**的文件夹下，执行如下命令：  
  
	chmod +x SwiftScript.sh  
  
它设置脚本文件的权限，使其能够被`shell`执行。  
  
####3、运行脚本  
运行脚本很容易，只需要指定脚本的名称，如下：  
  
	./SwiftScript.sh  
  
**./**是告诉shell，需要执行的脚本位于当前目录中，必须显式地指出这一点，否则shell会找不到脚本。  

该脚本运行后，你将看到该`shell`所在文件夹以及磁盘根文件夹的文件清单，你还将看到`Status = 0`，它指出用于显示文件命令运行正常，没有出现问题。  
  
####4、工作原理  
第一行是前面说过的`hash bang`，这里指定的应用路径为`/usr/bin/env`,它是一个为shell脚本设置环境的特殊命令，路径后面是一个你很熟悉的命令---启动`REPL`的命令[^1]：    
 
[^1]: REPL(Read-Eval-Print-Loop)是一个命令行工具，可用于快速尝试`Swift`代码，在`Mac OPX`中，可在应用程序`Terminal`中运行它。   
  
```   
  #!/usr/bin/env xcrun swift  

```  

下一条命令是一个`import`命令，与我们应用程序一样，`Swift`脚本也需要有基本的代码库才能运行。`Foundation`是向`Swift`脚本提供基本功能的框架，因此这里导入它：  
  
	import Foundation  
  
接下来是一个名为`Execution`的`Swift`类，其使命是执行命令，将需要执行的命令封装在类中，后面的执行将容易很多。  
  
	class Execution {
	
	}  
  
这个类中只有唯一的类方法`execute`，用关键字`class`定义：一个名为`path`的**String**参数(用于指定要运行的可执行文件的路径)以及一个名为`arguments`的**String**数组参数(这个参数是可选的，默认值是*nil*)。这个方法的返回值是一个`Int`值，指出了命令的执行状态。  
  
在`execute`方法中使用`Foundation`类的`NSTask`来设置启动路径和参数：  
  
	let task = NSTask()  
	task.launchPath = path
  
对于参数`arguments`，必须检查其为非**nil**才能使用`!`将其解包并赋值给`task`对象的属性`arguments`。  
  
	if arguments != nil {
		task.arguments = arguments!
	}

设置好`task`对象后，调用其`launch`命令来执行指定的命令：  
  
	task.launch()    

有关`NSTask`的文档指出，必须调用方法`waitUntilExit`让任务结束。  
  
	task.waitUntilExit()  
  
最后，将`task`对象的属性`terminationStatus`作为`Int`值返回（这个属性的类型为`Int32`，因此这里将其转换为`Int`，以方便调用者）：

	return Int(task.terminationStatus)  
  
在类定义的后面，定义了一个变量用于存储方法`execute`方法的返回值：  
  
	var status : Int = 0  
    
接下来，调用`execute`来执行命令`、bin/ls`（它显示目录中的文件）。  
另外，注意到一开始没有传入参数`arguments`：  
  
	status = Execution.execute(path: "/bin/ls")
	println("Status = \(status)")
  
接下来，再次调用方法execute来执行命令`/bin/ls`，但这次通过参数`arguments`指定了根路径`/`。
  
	status = Execution.execute(path: "/bin/ls", arguments:["/"])
	print("Status = \(status)")  
  
你可以使用`Swift`编写一个类，并在`Shell`脚本中使用。  
  
#####参考文献：  
Swift基础教程
