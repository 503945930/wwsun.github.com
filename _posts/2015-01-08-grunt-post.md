---
layout: post
title: Grunt, 基于JavaScript的项目构建工具
category: technique
---

由于前端工作中承担了越来越多的任务，使得现代Web开发中的前端变的足够复杂，因此应运而生的是对此类任务的自动构建工具。Grunt类似于`Make`或`Ant`，它可以完成诸如编译、单元测试等常用工作，从而帮助开发者进行自动重复的工作，节省开发者的时间。Grunt通常与我们[之前](http://wwsun.me/posts/bower-post.html)提到的`Bower`配合使用。

<!--more-->

## Grunt

### 基础知识

为什么要用Grunt？自动化你的任务，诸如代码最小化、代码编译、单元测试、代码规范校验等等重复的任务，你必须要做的工作越少，你的工作就变得越简单，从而避免重复人工操作中的误操作。Grunt同样依赖于Node，使用NPM进行安装。通过文件`Gruntfile`进行配置任务流程，这样你和你的团队就可以在之后非常简单的执行这些任务。

安装Grunt前，首先要确保你的Node和NPM处于新版本。你可以使用如下命令升级NPM：
	
	npm update -g npm

我们需要知道的是，Grunt包括三大组成部分：GruntJS CLI, GruntJS Task Runner, Grunt Plugins，分别代表Grunt的命令行工具、任务运行器、和插件库。

Grunt基本的三个概念分别为：Task, Target, Options

### 安装Grunt

为了能够在任何目录中都能使用grunt，首先你需要全局安装Grunt-cli，它是grunt的命令行工具，
安装完之后，你可以在命令行中直接使用grunt（安装过程可能需要你具备管理员权限）。

	npm install -g grunt-cli

这个过程会自动将`grunt`加入到你的path环境变量中。需要注意的是，
安装了grunt-cli并不会同时安装grunt。 cli的主要工作就是，当你运行grunt命令时，
CLI载入本地安装的Grunt库，应用`Gruntfile`中的配置项，执行你所请求的任务。
你可以通过[链接页面](https://github.com/gruntjs/grunt-cli/blob/master/bin/grunt)查看grunt的源代码。

为了能够使用grunt，接下来你需要创建项目的`package.json`清单文件，该文件用于描述该Node项目。
关于[package.json](https://docs.npmjs.com/files/package.json)的更多信息请参考相关文档。
然后将grunt添加到在`package.json`文件总的依赖列表中即可。

你需要在项目中安装`grunt`依赖，此时不需要全局安装！并且指定`--save-dev`，指明其实一个开发阶段的依赖项。

	npm install grunt --save-dev

### 创建`Gruntfile.js`文件

最后一步是创建`Gruntfile.js`文件，Grunt通过这个文件来载入可用任务以及配置相关参数。
下面的代码展示了一个最简单的`Gruntfile.js`文件：

	```javascript
	module.exports = function (grunt) {
		grunt.registerTask('default', []); // 注册一个默认的任务
	}
	```
	
值得关注的是，Grunt文件时一个普通的Node模块，遵循CommonJS模块化规范。

在上面的代码中，利用`grunt.registerTask`方法创建了一个名为`default`的默认任务，
当你在命令行中没有指定具体的任务的时候，默认的任务将会被执行。参数2则是一个任务列表，
你可以指定多个任务。

然后再项目的根目录执行如下命令即可运行grunt：

	grunt

### 设置Grunt任务

你可以通过安装插件来设置Grunt任务，并在代码中添加配置信息。Grunt插件通常都是作为npm的模块被发布的。
例如JSHint模块，你可以在Grunt中配置JSHint任务。但需要注意的是，你需要安装适用于Grunt的插件，
例如JSHint你需要安装的是`grunt-contrib-jshint`。

	npm install --save-dev grunt-contrib-jshint

你可以通过[Grunt的插件搜索](http://gruntjs.com/plugins)页面来检索你需要的插件。

接下来你要告诉Grunt你会使用的插件，并将其添加到指定的任务队列中，例如加入`default`任务队列。
现在`Gruntfile.js`的代码如下：

	```javascript
	module.exports = function (grunt) {
		
		// 配置任务
		grunt.initConfig({
			jshint: ['Gruntfile.js'],
		});
		
		// 每个插件都需要被逐个载入到Grunt中
		grunt.loadNpmTasks('grunt-contrib-jshint');
		
		grunt.registerTask('default', ['jshint']); // 注册一个默认的任务
	}
	```

### 使用Grunt来管理构建过程

- Development flow 开发流：Preprocessing, Liniting, Unit testing （关心的是开发效率）
- Release flow 发布流：Compilation, Image spriting, Bunding, Heavy testing, Minification, Perf. tuning （关心的是性能）
- Deployment flow 部署流：关心的是稳定性

## 其他的构建任务

### 将less转换成css

使用[LESS](http://lesscss.org/)可以让你的CSS代码遵守DRY原则，LESS的代码更易阅读，因为你可以对代码进行重用，
并且可以在CSS中使用变量。安装LESS插件：

	npm install grunt-contrib-less --save-dev
	
这个插件提供了一个名为`less`的任务，你可以在`Gruntfile.js`中载入它。

	grunt.loadNpmTasks('grunt-contrib-less');
	
`less`任务的配置对象如下：

	```javascript
	grunt.initConfig({
		less: {
			compile: {
				files: {
					'build/css/compiled.css': 'pubic/css/layout.less'
				}	
			}
		}
	})
	```

### 其他插件

我们通常会安装两个插件，分别是`load-grunt-tasks`[3]和`time-grunt`[4]。
第一个插件的作用是用来自动读取package.json文件中的`dependencies/ devDenpendencies/ peerDenpendencies`，
从而自动加载与给定模式匹配的Grunt任务。第二个插件的作用是在`console`窗口中显示任务的执行状态。

为了示例`Gruntfile.js`文件，我们还安装了官方的`grunt-contrib-copy`和`grunt-contrib-clean`文件用来进行文件的拷贝和清除。
借助这几个插件，我们编写的一个简单的`Gruntfile.js`文件如下：

	```javascript
	'use strict';
	
	module.exports = function (grunt) {
	    require('load-grunt-tasks')(grunt);
	
	    require('time-grunt')(grunt);
	
	    var config = {
	        app: 'app',
	        dist: 'dist'
	    };
	
	    grunt.initConfig({
	        config: config,
	        copy: {
	            dist: {
	                files: [
	                    {
	                        expand: true,
	                        cwd: '<%= config.app %>/',
	                        src: '*.html',
	                        dest: '<%= config.dist %>/',
	                        ext: '.min.html'
	                    }
	                ]
	            }
	        },
	
	        clean: {
	            dist: {
	                src: ['<%= config.dist %>/**/*'],
	                //filter: 'isFile' //only remove file, rather than folder
	                filter: function (filepath) {
	                    return (!grunt.file.isDir(filepath));
	                }
	            }
	        }
	    });
	};
	```

一个Gruntfile通常由下面几部分组成：

- 包装函数： `module.exports = function(grunt) { ... }`
- 项目和任务配置: `grunt.initConfig(...)`
- 加载的Grunt插件和任务
- 自定义任务


实际上，Grunt任务的书写方式可以有很多种，这里我们只是推荐一种最通用的书写方式。我
们在`initConfig`中定义了两个task，分别为`copy`和`clean`。更多的关于task的配置，请参考[2]

### Grunt的组件

- 任务 task： 通常一个任务需要一个插件来完成某项具体的构建工作
- 配置 configuration: 传递给`grunt.initConfig`的对象，每个Grunt任务都需要某些配置信息

## References

1. https://github.com/gruntjs/grunt-cli/blob/master/bin/grunt
2. http://gruntjs.com/configuring-tasks
3. https://github.com/sindresorhus/load-grunt-tasks
4. https://github.com/sindresorhus/time-grunt