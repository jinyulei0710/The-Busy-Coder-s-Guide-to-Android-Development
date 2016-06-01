###从Gradle创建Jar包

Gralde被用于Java开发已经有一段时间了，并且标准的用于Gradle的java插件知道如何创建JAR文件。

但是，我们使用的不是java插件。作为替代，我们使用的是android或android-library插件。在
后一种情况下，你可能提出它应该支持创建Jar包的任务，因为类库并没有使用资源等东西。不幸的是，
它没有，至少现在没有。因此，android或android类库项目中没有创建jar包的任务。

因为这些情形很常见，Jake Wharton就出来救场了。

在StackOverflow上Jake发布了一个提供应急的Gradle代码片段添加JAR包到android类库项目的回答。

      android.libraryVariants.all{  variant->
        def name = variant.buildType.name
        if(name.equals(com.android.builder.core.BuilderConstants.DEBUG)){
          return;
        }
        def task=project.tasks.create "jar${name.capitalize()}",Jar
        task.dependsOn variant.javaCompile
        task.from variant.javaCompile.destinationDir
      }

Groovy中的Gradle DSL主要是包含了构建数据结构。因为我们所有的构建变量会存在于android.libraryVariants
中。Jake的代码片段遍历了这些，筛掉那些针对debug构建的，并且会自动定义一个新的任务。新的任务会被命名
成jar...，...是构建类型的名称。他的代码片段然后会配置任务创建一个JAR文件，在代码编译之后，把结构
放在Java编译的目标路径。

把这个片段放在你的build.gradle文件的底部的最终结构是会添加一个像jarRelease的任务。注意，
jarRelease任务在你运行gradle tasks的时候没有出现，而是在你运行gradle tasks -all
获取完整列表的时候才会出现。

这没有创建一个完整的Jar包的artifact，所以如果你的计划是提交这个JAR到一个artifact仓库，
你会要做些额外的工作。但是创建手动发布的JAR包来说，它应该能胜任。
