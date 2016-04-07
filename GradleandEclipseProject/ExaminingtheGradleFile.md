###分析Gradle文件
这本书的样例代码包含了一个Gradle/Hello样例文件。这只是一个由Eclipse新项目向导创建的一个“hello，world”应用。

但是，它也包含了一个由Eclipse导出的build.gradle文件。

	buildscript{
 	 repostories{
 	   mavenCentral
  	}
  	dependencies{
  	  classpath 'com.android.tools.build:gradle:1.0.0'
  	}
	}
	apply plugin: 'com.android.application'

	dependencies{
   	compile fileTree(dir:'libs',include:'*.jar')
	}

	android{
 	 compileSdkVersion 19
 	 buildToolsVersion "21.1.2"
  	sourceSets{
  	  main{
      	   mainfest.srcFile 'AndroidManifest.xml'
        	 java.srcDirs=['src']
         	resource.srcDirs=['src']
         	aidl.srcDirs=['src']
         	renderscript.sirDirs=['src']
         	res.srcDirs=['res']
         	assets.srcDirs=['assets']   
    	}
    	//Move the tests to tests/java,tests/res,etc....
    	instrumentTest.setRoot('tests')
    
    	//Move the build types to build-types/
    	insrumentTest.setRoot('tests')
    
    	//Move the build types to build-tyes/
    	//For instance,build-types/debug/java,build-types/debug/	AndroidManifest.xml,...
    	//This moves them out of them default location under src//...which would
    	//confilct with src/ being used by the main source set.
    	//Adding new build types or product falvaors should be accompanied
    	//by a similar customization
    	debug.setRoot(‘build-type/debug’)
    	release.setRoot('build-typs/release')
 	 }
	}
	
这个文件中的大部分内容都已经在Gradle介绍那章涵盖到了。没有涵盖到的是sourcesets闭包－会在[紧接着的章回]()涵盖到。