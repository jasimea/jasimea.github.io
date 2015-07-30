---

layout: post
title: Better JavaScript Workflow With Jake
description: ""
modified: 2015-07-24
tags: [javascript, jake, nodejs, build automation]

---

### Introduction

Jake is a javascript build tool that can dramatically improve your javascript development workflow. It helps you to easily maintain and automate the development 
tasks such as compiling sass files, typescript, coffeescript, etc, concatenate and minify source code, optimising images and validating your javascript code with JSHint.
 Unlike [Grunt]("http://www.gruntjs.com", "Grunt") & [Gulp]("http://www.gulpjs.com", "Gulp"), jake is not a plugin based tool. 
You can create jake tasks with any valid javascript code.

In this blog post we will use jake to speed up the front end development process. We will look breifely what jake can do, before jumping into real development walk through.

###Setting up
In order to start with the jake, you need to have nodejs installed on your machine. If you know nothing about node js, visit the [download]("http:// www.nodejs.com", "Node JS") page and grab the installer for your operating system.Once nodejs installed, run this command in your terminal to install the jake.
{% highlight javascript %} 
   npm install -g jake
{% endhighlight %}
-g flag will install the jake glabally. To make sure jake has been installed properly, you can open the command line prompt and type  ``` jake --version ``` and it should output the current version of jake.


### The Jakefile
Every jake project has a file, ```jakefile.js``` which defines workflow for jake to execute. Jakefile is just executable javascript. You can include whatever the javascript you want in it. A task can be defined using ```task``` function. It has one required argument, the task name, and three optional arguments.

{% highlight javascript %} 
	task(name, [prerequirities], [action], opts);
{% endhighlight %}

The ```name``` argument is a string with the name of the task. And ```prerequisites``` is the optional array of the list of prerequisite to perform first. The ```action``` is a function defining the action to be executed for task. The ```opts``` is the normal javascript object for specifying any additional configurations to  the task. If you want to make the jake task asynchronous, set the async property to true and you task must call ```complete()``` to notify the jake that the task is done, and execution can proceed. By default ```async``` property is ```false```.

You can use ```desc(str)``` to add a description for task.

{% highlight javascript %}
   
	desc('This is the default task');
    task('default', function() {
        console.log('Jake is up and running...');
    });
    
    desc('This is the task with prerequisites.);
    task('build', ['clean', 'copy', 'compile'], function() {
        console.log('Clean, Copy & Compile tasks must be executed before entering to this task');
    });
    
    desc('This is an example of asyncronous task');
    task('test', { async: true }, function(param) {
		console.log(' You have passed ' + param);
        setTimeout(function() {
            complete();
        }, 10000);
    });
 
{% endhighlight %}

A Jake task can be executed as following:

{% highlight javascript %}
	jake [task][params]
{% endhighlight %}

For example, the ```test``` task from above code sample can be executed as:

{% highlight javascript %}
	jake test[paramValue]
{% endhighlight %}

If you do not specify a task name, then the default task will be executed.

### Setting up project.

Let's kickstart our app with angular-seed project. I recommend the angular-seed as it provides a great skeleton for bootstrapping. It also contains bunch of development and testing tools. To get started simple clone the angular-seed repository and install the dependencies.

{% highlight javascript %}
	git clone https://github.com/angular/angular-seed.git
{% endhighlight %}

#### Install dependencies

We have two types of dependecies in this project angular framework code and tools which helps to manage and test the application code. Tools can be downloaded via node package manager using the command ``` npm install ```.And angular framework code can be downloaded via ```bower``` using  the command: ``` bower install  ```. Angular team has preconfigured the ```npm`` to automatically install bower dependencies by running ```bower install ```command. So just need to install npm modules only which will install bower modules also.

After installing dependencies, you should find two new folders in your project.
	
-  ```node_modules``` - npm packages for tools
-  ```app\bower_components ```  - angular js framework files.

Angular-seed is preconfigured with simple webserver. And you can run the application with ```npm start ``` command. But it does not contain automated build system like Jake. 

### Jake In Action
Next you need to create a file called ```Jakefile.js``` .  This is where you will define and configure the tasks that you want jake to run. So our initial Jakefile will be like following:
``` javascript
var jake= require('jake');

desc('default jake task');
task('default', [], function() {
	console.log('Finished Building');
});
```

It's always a good practice to keep the build configuration separate from actual build file. Create a file called ```Jakefile.config.js``` .

``` javascript 
var Config = (function() {
	function JakeConfig() {
		this.source = 'app';
		this.build = 'generated/build';
		this.dist = 'generated/dist';
	}
	return Config;
})();

module.exports = new Config();
```
```configuration```  is a simple singleton module which contains list of properties like path, jshint options, etc. In this code source, build and dist are the path to the corresponding folders from current directory.

Next import the configuration in your jakefile.

```
 var jakeConfig = require('./jakefile.config.js');
```



#### Clean previous build
The clean task will remove the build artifacts from previous build. we use the ```jake.rmRf``` utility which recursively removes a directory and all its content.

``` javascript
desc('clean previous build  files');
task('clean', function() {
	console.log('cleaning the project....');
	jake.rmRf(jakeUtil.dist); // directory defined in the config file
	jake.rmRf(jakeUtil.build);
});
```
This will remove the ```generated/build``` and ```generated/dist``` directories and all its content.
#### Copy assets
After cleaning the project we need to copy the actual code & assets into the build directory. Node's built in file system api is too low level and too painful to use. So we will use the ```fs-jetpack``` api built on top of native file system API.  Visit [github](https://github.com/szwacz/fs-jetpack)  page for more details.

**Installation** 
``` 
npm install fs-jetpack --save-dev
``` 

**Usage**
``` javascript
var jetpack = require('fs-jetpack');
```

Our copy task will look like:

``` javascript 
task('copy', function (env) { 
	util.log('Copying assets...');		
	var source = jakeConfig.source,
		dest = jakeConfig.build,
		filesToInclude = [
			source + '/**/*.html',
			 source + '/scripts/**/*.js',
			 source + '/assets/**/*.css',
			 source + '/assets/images/**/*' 
		];
	if(env == 'dist') {
		source = jakeConfig.build;
		dest = jakeConfig.dist;
		filesToInclude = [
			source + '/assets/*',
			'*.!(css|js|html)'
		];
	}
	jetpack.copy(source, dest, { 
		matching: filesToInclude
	});
});
```
The copy task will do two tasks. Copy code and assets from source to build and from build to distribution folder. You need to pass the parameter env='list' in order to copy to list folder.

####Validating code with JSHint
#### Concatenating the Source file 
#### Minifying the source files
#### Compiling the less files
#### Configuring the Staging Server
####Wiredep the  bower dependencies
Wiredep is the node module used to wire the bower dependencies to your source code.  

Install module with npm.
``` npm install wiredep --save-dev ```

Install your bower dependencies
``` bower install jquery --save ```

Insert a placeholder in your app.html where your dependencies will be injected
``` html
	<html>
		<head>
			 <!-- bower:css -->
			 <!-- endbower -->
		</head>
		<body>
			<!-- bower:js -->
			<!-- endbower -->
		</body>
	</html>
``` 
Create the jake task as follows:
``` javascript 
var wiredep = require('wiredep');
//....
task('wiredep',  function() {
	wiredep({
		src: 'src/app.html',
		bowerJson: require('./bower.json'),
		directory: './bower_components'
	});		
});
```
##### How it works?
 Wiredep reads the dependencies array from your bower.json file, then reads the bower.json from each dependencies folder in your 
 bower_components folder. Based on these connection, it determines the order of scripts must be included before injecting them 
 between the placeholders in your source code.

If one of your dependencies does not have main in its bower.json, then you may want to override the default behaviour in your bower.son file like following:

``` javascript
{
	....
	"dependencies": {
		"package-without-main" : "0.0"
	},
	"overrides": {
		"package-without-main" : { 
			"main" : "package-main.js"
		}
	}
}
```

You can read more details about wiredep library [here](https://github.com/taptapship/wiredep).


#### Watch the filesystem for changes
####Minify HTML files with htmlmin
#### Minify the css files with CssMin
#### Implement the live reload
#### Wrapping up
#### Summary