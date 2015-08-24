---

layout: post
title: Better JavaScript Workflow With Jake
comments: true
description: ""
modified: 2015-07-24
tags: [javascript, jake, nodejs, build automation]
disqus_shortname: "jasimea"
disqus_identifier: "0608201501"
color: "#45b0e3"
---


Jake is a javascript build tool that can dramatically improve your javascript development workflow. It helps you to easily maintain and automate the development 
tasks such as compiling sass files, typescript, coffeescript, etc, concatenate and minify source code, optimising images and validating your javascript code with JSHint.
 Unlike [Grunt]("http://www.gruntjs.com", "Grunt") & [Gulp]("http://www.gulpjs.com", "Gulp"), jake is not a plugin based tool. 
You can create jake tasks with any valid javascript code.

In this blog post we will use jake to speed up the front end development process. We will look breifely what jake can do, before jumping into real development walk through.

### Setting up

---

In order to start with the jake, you need to have nodejs installed on your machine. If you know nothing about node js, visit the [download]("http:// www.nodejs.com", "Node JS") page and grab the installer for your operating system.Once nodejs installed, run this command in your terminal to install the jake.
{% highlight javascript %} 
   npm install -g jake
{% endhighlight %}
-g flag will install the jake glabally. To make sure jake has been installed properly, you can open the command line prompt and type  ``` jake --version ``` and it should output the current version of jake.

### The Jakefile

---


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

---

Let's kickstart our app with angular-seed project. I recommend the angular-seed as it provides a great skeleton for bootstrapping. It also contains bunch of development and testing tools. To get started simple clone the angular-seed repository and install the dependencies.

{% highlight javascript %}
	git clone https://github.com/angular/angular-seed.git
{% endhighlight %}

#### Install dependencies

We have two types of dependecies in this project angular framework code and tools which helps to manage and test the 
application code. Tools can be downloaded via node package manager using the command ```npm install```. 
And angular framework code can be downloaded via ```bower``` using  the command: ``` bower install```. 
Angular team has preconfigured the ```npm``` to automatically install bower dependencies by running ```bower install```command. 
So just need to install npm modules only which will install bower modules also.

After installing dependencies, you should find two new folders in your project.
	
-  ```node_modules``` - npm packages for tools
-  ```app\bower_components ```  - angular js framework files.

Angular-seed is preconfigured with simple webserver. And you can run the application with ```npm start ``` command. But it does not contain automated build system like Jake. 

### Jake In Action

---

Create a file called ```Jakefile.js``` in your project root directory. This is where you will define and configure tasks that you want jake to run. 
{% highlight javascript %}
	var jake= require('jake');
	desc('default jake task');
	task('default', [], function() {
		console.log('Finished Building');
	});
{% endhighlight %}

### Clean previous build

---

We need to clean/remove the build artifacts from previous build. ```jake.rmRf()``` is the  utility method which recursively remove the directories and all its content.

{% highlight javascript %}
desc('clean previous build  files');
task('clean', function() {
	console.log('cleaning the project....');
	jake.rmRf('dist');
});
{% endhighlight %}

This will remove the contents of ```dist/``` directory.

### Linting JavaScript code with JSHint

---
 
Linting is the process of analysing code for potential errors whith the use of programs. [JSHint](http://jshint.com/) is a static analysis tool for javascript. It analyzes JavaScript source code for common mistakes. Instead of using JSHint 
directly we  use ```simplebuild-jshint```, library that provides a simple interface to JSHint. Its convenient to use automation tools such as Grunt, Gulp or Jake.

Install JSHint and simplebuild-jshint npm modules using ```npm install``` commands as following

{% highlight javascript %}
	npm install -g jshint
	npm install --save-dev simplebuild-jshint
{% endhighlight %}

We need to validate source javascripts with JSHint, also we need to exclude the bower dependencies from validation.

#### JSHint Options
JSHint was designed to be very configurable. These configuration options can be found [here](http://jshint.com/docs/options/). Define the JSHint rules in 
jakeConfig file like following:

{% highlight javascript %}
	this.jsHintOptions = {
		curly: true,
		camelCase: true,
		window: false,
		document: false,
		setTimeout: false,
		quotmark: 'single'
	};
{% endhighlight %} 

Our lint task is as follows:

{% highlight javascript %}
	var jshint = require("simplebuild-jshint");

	task('jshint', function () {
	    jshint.checkFiles({
	        files: [
	            "app/**/*.js",
				"!app/**/*_test.js",
            	"!app/bower_components/**/*.js"
        	],
        	options: {
            	curly: true,
            	quotmark: 'single',
            	eqeqeq:true
        	}
    	}, complete, fail);
	}, { async: true });

{% endhighlight %}

Task will lint the all javascript inside app folder, it also excludes the bower_component folder from liniting.

### Wiredep the  bower dependencies

---

After bower packages has been installed, you could add them manually to your application's HTML file. To avoid this manual intervention, you can use wiredep
to automate this process. Wiredep enables you to wire Bower dependencies into your source code.


Install module with npm.
{% highlight javascript %}
	 npm install wiredep --save-dev
{% endhighlight %}

Edit app/app.html and add placeholders for use by wiredep, as shown below:
{% highlight html %}
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
{% endhighlight %}

Create the jake task as follows:

{% highlight javascript %} 
var wiredep = require('wiredep');

desc('Wiring bower dependencies...');
task('wiredep', function () {
	console.log('Wiring bower dependencies...');
    wiredep({
        src: 'app/index.html', 
		bowerJson: require('./bower.json'), 
		directory: './bower_components'
    });
});
{% endhighlight %}

#### How it works?
 Wiredep reads the dependencies array from your bower.json file, then reads the bower.json from each dependencies folder in your 
 bower_components folder. Based on these connection, it determines the order of scripts must be included before injecting them 
 between the placeholders in your source code.

If one of your dependencies does not have main in its bower.json, then you may want to override the default behaviour in your bower.son file like following:

{% highlight javascript %}
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
{% endhighlight %}

You can read more details about wiredep library [here](https://github.com/taptapship/wiredep).

### Copy assets to build folder

---
	
 Copy task will copy the assets and source code from source folder to build folder. Node's built in file system api 
 is very low level and because of that often painful to use. We use ```fs-jetpack``` module, which gives more convenient API to work with file system.
 Visit [github](https://github.com/szwacz/fs-jetpack)  page for more details.

**Installation** 
{% highlight javascript %}
npm install fs-jetpack --save-dev
{% endhighlight %}
	 
Our copy task will look like:

{% highlight javascript %}
var jetpack = require('fs-jetpack'); 
task('copy', function () { 
	var source = 'app', 
		dest = 'dist',
		filesToInclude = ['**/*.html', 'assets/*', '*.!(css|js|less)' ];
	jetpack.copy(source, dest, { matching: filesToInclude });
});

{% endhighlight %}

### Compiling the less files

---

We need to compile less files to standard css files. Less can be used on command line via npm, or  as a script file for runtime 
compilation inside browser itself, or via wide variety of third party tools. Here we use less via command line. 
```jake.exec``` command is the best option with jake js for executing the shell commands.

Install less css via npm ``` npm install -g less ```. ***If you don't want to install less as global module,
then you should change the less command relative to your node_modules folder***.

{% highlight javascript %}
	task('less', function () {
		var list = new jake.FileList();
		list.include('app/**/!(_)*.less');

		list.forEach(function (file) {
			var dest = file.substring(file.indexOf("/"), file.lastIndexOf('.')) + '.css';
			jake.exec('lessc ' + file + ' > ' + dest, { printStdout: true }, function () {
				complete();
			});
		});
	}, { async: true });
{% endhighlight %}

In above code ```jake.FileList``` searches the filesystem and find files into arrray. It takes list of glob-patterns and filenames as parameter, and lazily
creates a list  of files to include. More on [FileList]("http://jakejs.com/docs#file_list").

### Optimizing the application

---

The best way to optimize our application is to reduce the number of requests by combining multiple files as few as possible and to reduce the size of files with the help of technique like minification. 
In this section we will check how we can combine our application source code into a single JavaScript file and also minify source files to reduce the size.

Let's optimize our application with usemin-cli, which replaces reference from non-optimized scripts,stylesheet and other assets to their
optimized version within a set of HTML files.

Usemin blocks can be expressed as following:
{% highlight html %}
<!-- build<type>(alternate search path) <path> -->
... List of script/link tags goes here.
<!-- endbuild -->
{% endhighlight %}

Change our app/index.html like s following:

{% highlight html %}
<!-- build:js scripts/vendor.js -->
<!-- bower:js -->
<!-- endbower -->
<!-- endbuild -->
<!-- build:js scripts/app.js -->
<!-- app:js -->
<script src="app.js"></script>
<script src="view1/view1.js"></script>
<script src="view2/view2.js"></script>
<script src="components/version/version.js"></script>
<script src="components/version/version-directive.js"></script>
<script src="components/version/interpolate-filter.js"></script>
<!-- endapp -->
<!-- endbuild -->
{% endhighlight %}

After running this task, all our bower dependencies will be concatenated and minified into a single file which will be place into scripts/vendor.js in dist
folder. Also scripts/app.js will contain our application script.

Install usemin-cli:

{% highlight javascript %}
npm install usemin-cli --save
{% endhighlight %} 

Optimize task look like this:

{% highlight javascript %}
task('optimize', { async: true }, function () {
	jake.exec('node ./node_modules/usemin-cli/bin/usemin app/index.html -d dist -o dist/index.html', 
	{ stdout: true }, function () {
        complete();
    });
});
{% endhighlight %}

### Configuring the Staging Server

---

This task allows you to create a http server. Once run this task, you can access the application on http://localhost:8080. We use ```http-server``` module
which is very simple and powerful. Install ```http-server```  module via npm.  

{% highlight javascript %}
task("server", ["build"], function () {
    jake.exec("node ./node_modules/http-server/bin/http-server " + "dist", {
        interactive: true
    }, complete);
}, { async: true });
{% endhighlight %}

### Watch the filesystem for changes

---

Watch task will watch your app directory, and if any file changes, jake will automatically rerun build task. Install ```nodemon``` module via npm, which will
monitor for any changes.

{% highlight javascript %}

var nodemon = require('nodemon');

task('watch', function () {
    nodemon({
        ext: "sh bat json js html css hbs less scss",
        ignore: ["build", "dist"],
        exec: "jake build",
        execMap: {
            sh: "/bin/sh",
            bat: "cmd.exe /c",
            cmd: "cmd.exe /c"
        }
    }).on("restart", function (files) {
        console.log("*** Restarting due to", files);
    });
});
{% endhighlight %}

Nodemon accepts several configuration options. ```ext``` says the extensions of files thats should be watched, ```ignore``` is the list directories or files
to be excluded from watching, and ```exec``` is the command that should be executed when some changes occur in filesystem. You can find more about nodemon [here]("https://github.com/remy/nodemon").  


### Wrap up
---

Our build task put all the task togethere into one task. You can start building the application with ``` jake watch``` which will call the build task and 
starts watching application.

{% highlight javascript %}
task('build', [ 'clean', 'jshint', 'wiredep', 'copy', 'less', 'optimize'], function () {
   console.log('Finished building successfully!');
});
{% endhighlight %}

It's up to your choice, whether to use Jake over grunt, gulp or any other build libraries. Each of these build tools have their own pros and cons. But jake 
provides a clean JavaScript syntax which is more readable and easy to maintain. As jake  is not a plugin centric tool, it can even be used without any other dependencies on it.

You can find the full source code here in [Github](http://jasimea.github.io).