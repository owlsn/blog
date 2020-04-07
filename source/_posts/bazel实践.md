title: bazel实践
date: 2019-11-30 16:07:58
tags: [cpp]
categories: 学习笔记
about:
---
之前总结了`c++`开发的一些基础，现在总结一下使用`bazel`做c++项目构建工具实践中的一些注意点。   
### helloworld   
官方提供了几个`helloworld`的构建例子，这里简单过一下项目结构，
`git clone https://github.com/bazelbuild/examples`,写好BUILD文件后，执行bazel对应的命令即可完成构建   
<!-- more -->

		examples
		└── cpp-tutorial
		    ├──stage1
		    │  ├── main
		    │  │   ├── BUILD
		    │  │   └── hello-world.cc
		    │  └── WORKSPACE
		    ├──stage2
		    │  ├── main
		    │  │   ├── BUILD
		    │  │   ├── hello-world.cc
		    │  │   ├── hello-greet.cc
		    │  │   └── hello-greet.h
		    │  └── WORKSPACE
		    └──stage3
		       ├── main
		       │   ├── BUILD
		       │   ├── hello-world.cc
		       │   ├── hello-greet.cc
		       │   └── hello-greet.h
		       ├── lib
		       │   ├── BUILD
		       │   ├── hello-time.cc
		       │   └── hello-time.h
		       └── WORKSPACE

从这几个`helloworld`的例子里，基本就能学会构建简单的项目，不过这几个例子里没有讲到引用第三方库的情况
#### 1.单构建目标
srcs说明源文件的相对路径,`cc_binary`表示构建为可执行二进制文件

		cc_binary(
		    name = "hello-world",
		    srcs = ["hello-world.cc"],
		)
#### 2.多构建目标
`deps`表示`hello-world`依赖`hello-greet`库,`cc_library`将`hello-greet`构建为`library`供`hello-world`链接,hdrs指明头文件，因为在同一目录下，所以直接写文件名就可以找得到对应的文件

		cc_library(
		    name = "hello-greet",
		    srcs = ["hello-greet.cc"],
		    hdrs = ["hello-greet.h"],
		)
		
		cc_binary(
		    name = "hello-world",
		    srcs = ["hello-world.cc"],
		    deps = [
		        ":hello-greet",
		    ],
		)
#### 3.跨目录构建
最终构建的可执行文件为`hello-world`,需要链接`hello-time`和`hello-greet`库文件，`hello-time`源代码文件在`lib`目录下   
对于`lib/BUILD`文件,构建`hello-time`库，要设置`visibility`属性,这样才能被`hello-world`引用到, `visibility`属性默认是`private`，具体的规则`//`表示向外层目录找到的第一个`WORKSPACE`文件所在的目录，然后`main`表示`main`文件夹,`//main:__pkg__`表示的是`main`文件夹下的包都可以看到这条`rule`，`rule`表示的就是整个`cc_library`表示的内容，也就是可以引用`hello-time`库

		cc_library(
		    name = "hello-time",
		    srcs = ["hello-time.cc"],
		    hdrs = ["hello-time.h"],
		    visibility = ["//main:__pkg__"],
		)
对于`main/BUILD`文件，不同的地方就是`deps`里增加了引用`hello-time`的内容，`//lib:hello-time`表示`lib`文件夹下的`hello-time`库,如果`hello-time`的构建规则里没有`visibility`那一项，这里引用就会报错

		cc_library(
		    name = "hello-greet",
		    srcs = ["hello-greet.cc"],
		    hdrs = ["hello-greet.h"],
		)
		
		cc_binary(
		    name = "hello-world",
		    srcs = ["hello-world.cc"],
		    deps = [
		        ":hello-greet",
		        "//lib:hello-time",
		    ],
		)
#### 4.引用目标库的路径规则   
		//path/to/package:target-name 
1.如果`target-name`是`rule target`,`path/to/package`就是到`BUILD`文件目录的路径，`target-name`就是`BUILD`文件里定义的需要编译的库   
2.如果`target-name`包含路径的target，那么`path/to/package`就是到`BUILD`的根`WORKSPACE`文件的路径，`target-name`路径最后一项表示构建目标   
3.当在项目根目录内执行构建的时候，可以直接使用`//`表示根目录   
4.当在`BUILD`文件目录内执行的时候，可以直接使用`:target-name`表示构建目标   
#### 5.构建外部依赖   
`bazel`项目一般来说一个`WORKDPACE`文件就标志一个独立的项目，也可以说是一个独立的模块，当要引用其他项目中的库的时候，有几种情况
##### 构建本地bazel项目
引用`coworkers_project`中的库，在`my_project/WORKSPACE`中定义`local_repository`, `coworkers_project`中的项目`//foo:bar`，在`my_project`的BUILD中可以通过`@coworkers_project//foo:bar`方式引用

		local_repository(
		    name = "coworkers_project",
		    path = "/path/to/coworkers-project",
		)
##### 构建本地非bazel项目
在`my_project/WORKSPACE`中使用`new_local_repository`创建本地非`bazel`构建的项目，另外需要自己写一个`BUILD`文件给`coworkers_project`定义编译的规则

		new_local_repository(
		    name = "coworkers_project",
		    path = "/path/to/coworkers-project",
		    build_file = "coworker.BUILD",
		)
在`coworker.BUILD`文件中，就是构建`coworker`库的相关的内容。这个`BUILD`文件的路径，是相对于写`new_local_repository`的`WORKSPACE`的文件的，然后就可以通过`@coworkers_project//:some-lib`的方式引用`some-lib`库

		cc_library(
		    name = "some-lib",
		    srcs = glob(["**"]),
		    visibility = ["//visibility:public"],
		)
##### 构建远程项目
构建远程项目的时候一般需要先加载`bazel_tools`中获取远程资源的方法，比如`http_archive`和`git_repository`，分别通过`http`的方式获取和`git`的方式获取
1. `http_archive`方式   
其实和构建本地项目类似

		load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

		http_archive(
		      name = "my_ssl",
		      urls = ["http://example.com/openssl.zip"],
		      sha256 = "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
		      build_file = "@//:openssl.BUILD",
		  )
2. `git_repository`和`new_git_repository`方式  
同`http_archive`方式，只是有些参数不太相同
3. 缓存  

		ls $(bazel info output_base)/external
可以查看通过引用外部依赖的方式加载的项目，比如上面说的几种方式   
### 最佳实践
这里通过构建具体项目来熟悉`bazel`的用法，这里主要写两个，一个是`bazel`构建项目，一个是非`bazel`构建项目   
项目结构

		dir
		└── project
		    ├──WORKSPACE
		    ├──main
		    │  ├── helloworld
		    │  │   ├── BUILD
		    │  │   └── hello-world.cc
		    │  └── WORKSPACE
		    ├──common
		    │  ├── libs
		    │  │   ├── BUILD
		    │  │   ├── hello-greet.cc
		    │  │   └── hello-greet.h
		    │  └── WORKSPACE
		    └──thirdparty
		       ├── googletest
		       │   ├── BUILD
		       │   ├── ......
		       ├── boost
		       │   ├── boost.BUILD
		       │   ├── boost.bzl
		       │   ├── ......
		       └── WORKSPACE
#### googletest
1.其中`googletest`本身就是`bazel`构建的项目，所以直接将`googletest`项目源码从git上面clone下来，放到`thirdparty`中，如果在`common`或者`main`中需要引用`googletest`中的`test_main`库，首先需要在`common`或者`main`的`WORKSPACE`文件中将`thirdparty`定义为`local_repository`,而项目的`path`参数需要根据实际的情况来填写，可以填写相对路径,然后就可以在`main`或者`common`中引用`googletest`中的库了   
2.上面是单独饮用googletest库的操作，但是对于模块化来说，整个`thirdparty`应当看作是一个整体，所以可以将`thirdparty`下的项目先在`thirdparty`文件夹下添加WORKSPACE文件，将之定义为模块，然后在`common`和`main`中，将整个`thirdparty`定义为local_repository，而引用thirdparty下的某个其他库，就可以以相对路径的形式引用   
3.这样引用之后，在hello-world源代码中，可以直接饮用gtest头文件

		#include "gtest/gtest.h"
#### boost
1.boost和googletest不同，因为boost本身不是bazel构建的项目，如果我们想要和引用gtest一样引用boost，就需要我们自己编写编译boost的BUILD文件   
2.编写BUILD的文件的时候，可以自定义处理函数，将这些自定义函数放在.bzl的文件中，然后通过类似于引用bazel_tool库的方式引用里面的方法   

		load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")
这里如果自己定义了一些处理方法,然后放在boost.bzl文件中，那么就可以以下面的方式引入

		load("@workspace_name//path/to/boost:boost.bzl", "some-function")
通过这种方式可以简化BUILD文件里面的一些编写规则   
3.然后就可以和引入googletest中的库类似的方式引入boost中的库了
#### 备注
1.其实可以将整个thirdpary中的库都以http_archive的方式或者git_repository的方式代替local_repository的方式加载，将源码下载在本地后加载的方式和在线获取外部依赖的方式相比，可能会适用于有些不能在线获取资源的情况，而且将第三方资源和本地代码放在一起会直观一些
### 参考资料
[1].[Introduction to Bazel: Building a C++ Project](https://docs.bazel.build/versions/master/tutorial/cpp.html)   
[2].[c/cpp rules](https://docs.bazel.build/versions/master/be/c-cpp.html)   
[3].[common.visibility](https://docs.bazel.build/versions/master/be/common-definitions.html#common.visibility)   
[4].[Working with external dependencies](https://docs.bazel.build/versions/master/external.html)   
[5].[http_archive](https://docs.bazel.build/versions/master/repo/http.html#http_archive)   
[6].[git_repository](https://docs.bazel.build/versions/master/repo/git.html#git_repository)