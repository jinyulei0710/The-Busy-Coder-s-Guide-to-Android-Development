###关键构建相关任务

为了找出什么任务是你可用的，你可以从项目目录中运行gradle任务。这就会导致类似如下的输出：

	:tasks

	----------------------------------------------------

	All tasks runnable from root project

	--------------------------------

	Android tasks

	-------------

	androidDependencies - Displays the Android dependecies of the project
	signingReport - Displays the signing info for each vairiant

	Build tasks

	---------------
	assemble - Assembles all variants of all applications and secondary 	packages.
	assembleDebug-Assemables all Debug builds
	assembleDebugTest - Assembles the Test build for the Debug build
	assembleRelase - Assembles all Release builds 
	build - Assembles and tests this project
	buildDependents - Assembles and tests this project and all projects that 	depend on it.
	buildNeeded - Assembles and tests this project and all project it depends on.
	clean - Deletes the build directory

	Build Setup tasks

	------------------
	init - Initializes a new Gradle build.[incubating]
	wrapper - Generates Gradle wrapper files.[incubating]

	Help tasks

	------------------
	components-Displays the components produced by the root project 'Decktastic'.[incubating]
	dependencies-Displays all dependencies declared in root project 'Decktastic'.
	dependencyInsight-Displays the insight into a specific dependency in root project 'Decktastic'
	help-Displays a help message
	projects-Displays the sub-projects of root project 'Decktastic'
	projecties-Displays the properties of root project 'Decktastic'
	tasks-Displays the tasks runnable from root project 'Decktastic'

	Install taks

	-------------
	installDebug-Installs the debug build
	installDebugTest-Installs and runs the tests for Build 'debug' on connected devices.
	connectedCheck - Run all device checks using Device Provider and Test Servers.
	lint - Runs lint on the Debug build
	lintDebug -Runs lint on the Debug build
	lintRelease - Runs lint on the Release build

	Other tasks

	------------
	compileDebugSource
	compileDebugSource
	compileReleaseSource

	Rules 

	-----------
	Pattern:clean:Cleans the output files of a task.
	Pattern:build:Assembles the artifacts of a configuration
	Pattern:upload:Assembles and uploads the artifacts belonging to a configuration.

	To see all tasks and more detail,run with --all.

	BUILD SUCCESSFUL

	Total time:9.669 secs
	
这个列表是根据build.gradle的内容自动生成的，尤其是包含由com.android.application插件定义的任务。

原则上说，你是假定在运行任务的时候详述整个任务名的。但是，你可以简略的表述方法，只要它能唯一地对任务进行识别。

开发者会使用的最常见的任务，至少短期来说，是installDebug.这会构建一个调试模式的应用并将其安装在一个可用的设备或模拟器上。这大致是跟基于ant命令行构建的ant install debug对应的。

只要有installDeubg，那就有installRelease.任务的Debug和Release部分不是硬编码的，而是源于定义在build.gradle文件中的"build types"。概念以及规则和用法会在下一章涵盖到。但是，默认情况下installRelease是没有的，因为安装应用需要的是签名过的APK,并且给Android的Gradle不知道如何对它进行签名。同样我们会在下一章讲到。

如果你仅仅想要构建应用，却不安装它,assembleDebug(aD)或assmbleRelease(aR)会完成这个目标.如果你要从真机或模拟器上卸载应用,unintallDebug（uD）和uninstallRelease(uR)应用有用。

至于其它任务，例如“check”任务，会在之后的章节涵盖到。