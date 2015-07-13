# Better JavaScript Workflow With Jake

Jake is a javascript build tool that can dramatically improve your javascript development workflow. It helps you to easily maintain and automate the development tasks such as compiling sass files, typescript, coffeescript, etc, concatenate and minify source code, optimising images and validating your javascript code with JSHint.

In this blog post we will use jake to speed up the front end development process. We will look breifely what jake can do, before jumping into real development walk through.


### Introduction
Unlike [Grunt]("http://www.gruntjs.com", "Grunt") & [Gulp]("http://www.gulpjs.com", "Gulp"), jake is not a plugin based tool. You can create jake tasks with any valid javascript code.


###Setting up
In order to start with the jake, you need to have nodejs installed on your machine. If you know nothing about node js, visit the [download]("http:// www.nodejs.com", "Node JS") page and grab the installer for your operating system.Once nodejs installed, run this command in your terminal to install the jake.
```
   npm install -g jake
```
-g flag will install the jake glabally. To make sure jake has been installed properly, you can open the command line prompt and type  ``` jake --version ``` and it should output the current version of jake.


### The Jakefile
Every jake project has a file, ```jakefile.js``` which defines workflow for jake to execute. Jakefile is just executable javascript. You can include whatever the javascript you want in it. A task can be defined using ```task``` function. It has one required argument, the task name, and three optional arguments.
``` 
task(name, [prerequirities], [action], opts);
```
The ```name``` argument is a string with the name of the task. And ```prerequisites``` is the optional array of the list of prerequisite to perform first. The ```action``` is a function defining the action to be executed for task. The ```opts``` is the normal javascript object for specifying any additional configurations to  the task. If you want to make the jake task asynchronous, set the async property to true and you task must call ```complete()``` to notify the jake that the task is done, and execution can proceed. By default ```async``` property is ```false```.

You can use ```desc(str)``` to add a description for task.

``` javascript
    desc('This is the default task');
    task('default', function() {
        console.log('Jake is up and running...');
    });
    
    desc('This is the task with prerequisites.);
    task('build', ['clean', 'copy', 'compile'], function() {
        console.log('Clean, Copy & Compile tasks must be executed before entering to this task');
    });
    
    desc('This is an example of asyncronous task');
    task('test', { async: true }, function() {
        setTimeout(function() {
            complete();
        }, 10000);
    });
    
```
You can run these task like:
``` javascript 
jake [task][params]
```
For example, you can run the parameterized task like ``` jake taskWithParam [paramValue1,paramValue2]```. If you do not specify the task name, then the default task will be executed.

### Setting up project.
To get started the real development process, simply clone the angular seed project into your local system. Angular-Seed is an application skeleton for typical angular js web app. Clone the angular seed applicatio  using git:

``` 
git clone https://github.com/angular/angular-seed.git
```

#### Install dependencies
We have two kinds of dependencies in this project tools and framework. Tools are the node JS packages help us to maintain and test our application. It can be installed via ```npm``` using command :

``` 
npm install
```
When you issue this command, the npm utility will start to grab the dependencies listed in your package.json > dependencies/devDependencies section from the central npm repository.  In same way you can install the client side framework dependencies with bower package manager using command:
```
bower install
```
Similar to npm, bower is a package management tool for client side libraries and assets. It keeps track of these packages in a manifest file, ``` bower.json```. You can visit bower.io to get more details about it.

You should find two new folders in your project:

 - node_modules - contains the npm packages for the tools we need.
 - app/bower_components -  contains front end libraries like angular and jQuery.

### Automating the process

*Regular javascript workflow - expand this part*

*use compress & minify*

1.	We need to clean the content of build directory before initiating new build.
2.	We need to compile our less files to standard css files.
3.	We need to validate the JavaScript code against best practices using JSHint.
4.	We need to minify and compress the images.
5.	We need to concatenate & compress all the javascript source code into single file.
6.	Concatenate and minify all the compiled css files into a single file.
7.	minify and optimize the html files
8.	Create a staging http server
9.	livereload 
10.	Watch the application for changes and do all the above steps whenever we change the system
11.	deploy the staging and production version to azure web services / amazon a3
12.	automatically change the version number and tag production releases in my git repository. 

### Let's get started 

### Clean previous build




