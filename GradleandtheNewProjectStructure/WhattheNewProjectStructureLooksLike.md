###新的项目结构是什么样的

有了这些知识背景，让我们看下“Gradle／helloNew”这个样例项目。这个项目开始是个Eclipse项目，
然后通过Eclipse导出向导有了个build.gradle文件。尽管之后，它会被重组成适应新的项目结构。

####目录树
预先组织好的项目目录树是相当传统的，只是添加了一些Gradle特性文件：

	Hello
	|--AndroidManifest.xml
	|--assets/
	|build.gradle
	|libs/
	|    |--android-support-v4.jar
	|--  local.properties
	|--  proguard-project.txt
	|--  project.properties
	|--  res/
	|--  |drawable-hdpi/
	|    |--drawable-hdpi/
	|    |   |--ic_launcher.png
	|    |-- drawable-ldpi
	|    |-- drawable-mdpi
	|    |   |--ic_launcher.png
	|    |-- drawable-xhdpi
	|    |   |--ic_launcher.png
	|    |--layout/
	|    |   |--activity_main.xml
	|    |--menu
	|    |    |--main.xml
	|--  |values/
	|--  |-- dimens.xml
	|--  |-- strings.xml
	|--  |-- styles.xml
	|-- values-sw600dp/
	|    |--dimens.xml
	|-- values-sw720dp-land/
	|    |--dimens.xml
	|-- values-v11/
	|    |--styles.xml
	|-- values-v14
    |--styles.xml
	|--src/
   	 |-- com/
        	|--commonsware/
          	  |--android/
              	  |--gradle/
                    	|--hello/
                      	  |--MainActivity.java


（注意：以上列表只包含了当前讨论的相关内容)

新的项目结构，就有点不同了：


	HelloNew

	|-- build.gradle
	|-- libs/
	|    |--android-support-v4.jar
	|--  local.properties
	|--  proguard-project.txt
	|--  project.properties
	|--  src／
    	|-- main/
      	  |--AndroidManifest.xml
       	 |--assets/
       	 |--java/
        	|  |--com/
        	|     |--commosware/
        	|       |--android/
        	|           |--gradle/
       	    |               |--hello/
         	|                              |--
      MainActivity.java
        |-res/
            |--  |drawable-hdpi/
            |    |--drawable-hdpi/
            |    |   |--ic_launcher.png
            |    |-- drawable-ldpi
            |    |-- drawable-mdpi
            |    |   |--ic_launcher.png
            |    |-- drawable-xhdpi
            |    |   |--ic_launcher.png
            |    |--layout/
            |    |   |--activity_main.xml
            |    |--menu
            |    |    |--main.xml
            |--  |values/
            |--  |-- dimens.xml
            |--  |-- strings.xml
            |--  |-- styles.xml
            |-- values-sw600dp/
            |    |--dimens.xml
            |-- values-sw720dp-land/
            |    |--dimens.xml
            |-- values-v11/
            |    |--styles.xml
            |-- values-v14
                |--styles.xml
虽然libs目录和build.gradle和相关构建文件还在它原来的位置，其它的都变了。

在新的项目结构中，src／是源代码集的根目录，不仅仅是存放源代码的地方。有一个叫作“main”的源集在src/main目录中。在这个目录中，有assets/和res/目录以及AndroidManifest.xml文件。并且呢，有一个包含了了java源代码树的java/目录（本来放在原来的src／目录中）。

####build.gradle文件
build.gradle文件就跟我们在Gradle介绍那一章描述的很像：

	buildscript{
  	  repositories{
     	   mavenCentral()
    	}
    	dependencies{
       	 classpath 'com.android.tools.build:gradle:1.0.0'
    	}
	}

	apply plugin: 'com.android.application'

	dependencies{
	}

	android{
   	 compileSdkVersion 19
    	buildToolsVersion "21.1.2"
	}    

我们有来描述我们构建工具所需的buildscript闭包，com.android.application插件。我们所编译的android版本以及使用的构建工具版本。

作为结果呢，我们有了标准的任务，包括installDebug.
