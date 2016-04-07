###再访旧式的Gradle文件

鉴于这一切，让我们再次访问从Eclipse导出的build.gradle文件。

	buildscript{
    	repositories{
       	mavenCentral
    	}
    	dependencies{
      	 classpath 'com.android.tools.build:gradle:1.0.0'
    	}
	}

	apply plugin: 'com.android/application'

	dependencies{
  	 compile fileTree(dir:'libs',include:'*.jar')
	}

	android{
    	compileSdkVersion 19
    	buildToolsVersion "21.1.2"
    
    	sourceSets{
      	 main{
         	 manifest.srcFile 'AndroidManifest.xml'
          	java.srcDirs=['src]
       	}
    	}
	}

导出的build.gradle文件中大量android相关的配置在android闭包的sourceSets闭包中。在这里，我们指定什么应该被编译。

Gradle让当前分开的全部扔到src/中，包括java代码，AIDL接口定义RenderScript源文件，以及java形式的资源（不要和android资源混淆）。因为这个构建文件是给传统的目录结构的，这些类型的输入文件依然会在src/目录中，所以main源集把android指向每一个的src目录。此外，main指示了minifest文件的名称以及res/和asssets目录树的位置。

setRoot()调用储备的debug和release构建类型来指示它们的源集。如果它们存在，应该在分开的build-types/目录中，就跟被java源代码使用的src/位置一样。