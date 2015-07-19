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
When you issue this command, the npm utility will grab the dependencies listed in your package.json from the central npm repository.  Similarly you can install the client side framework dependencies with bower package manager using command:
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
10.	test the application 
10.	Watch the application for changes and do all the above steps whenever we change the system
11.	deploy the staging and production version to azure web services / amazon a3
12.	automatically change the version number and tag production releases in my git repository. 

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

``` javascript
task('jshint', function () {
    var options = {
        bitwise: true,
        curly: false,
        eqeqeq: true,
        forin: true,
        immed: true,
        latedef: false,
        newcap: true,
        noarg: true,
        noempty: true,
        nonew: true,
        regexp: true,
        undef: true,
        strict: true,
        globalstrict: true,
        trailing: true
    };

    process.stdout.write('Linting javascript code...');

    var files = jetpack.find('app', { matching: ['*.js', '!**/*_spec.js'] }).forEach(function (f) {
        var content = fs.readFileSync(f);
        var pass = jshint(content, options, options);
        console.log(jshint.errors);
    });
}, { async: true });
```

#### Concatenating the Source file 

``` javascript
	
	
var bowerDependenciesRegEx = /(([ \t]*)<!--\s*bower:js\s*-->)(\n|\r|.)*?(<!--\s*endbower\s*-->)/gi;
var appDependenciesRegEx = /(([ \t]*)<!--\s*app:*(\S*)\s*-->)(\n|\r|.)*?(<!--\s*endapp\s*-->)/gi

task('concat', function() { 
	var htmlContent = fs.readFileSync('app/index.html');
	
	jetpack.append('app/vendor.min.js', concatScriptDependecies(htmlContent.toString(), bowerDependenciesRegEx));	
    jetpack.append('app/app.min.js', concatScriptDependecies(htmlContent.toString(), appDependenciesRegEx)); 	

	htmlContent = htmlContent.toString().replace(bowerDependenciesRegEx, '<script src="vendor.min.js"></script>');
	htmlContent = htmlContent.toString().replace(appDependenciesRegEx, '<script src="app.min.js"></script>');
	
	jetpack.write('app/app.build.html', htmlContent);
});

function concatScriptDependecies(content, regEx) {
    
    var regexMatch = content.toString().match(regEx);
    var concat = "";
    
    var scriptTags = regexMatch.join('').toString().match(/<script.*src=['"]([^'"]+)/gi);
    scriptTags.forEach(function(tag) {
        var path = tag.toString().match(/src\s*=\s*['"]([^"']+)/)[1];
        concat += jetpack.cwd('app').read(path);
    });
    
    return concat;
}

```

#### Minifying the source files
We use Uglify JS 2 for compressing and minifying our javascript code. install node.js 
module using npm.

``` npm install --save-dev uglify-js ```

 Uglify js is providing commandline utility to minify the source files. But we want to use its code api.

``` javascript
	var uglifyjs = require('uglify-js');
	//....
	task('minify-scripts', function(){
		var scripts = [
			dist + '/scripts/app.js',
			dist + '/scripts/vendor.js'
		];
		scripts.forEach(function(file) {
			console.log('Minifying ' + file);
			var result  = uglifyjs.minify(file);
			jetpack.write(file, result.code);
		});	 
	});
	 
```


#### Compiling the less files

``` javascript
	task('less', function () {
    var lessfile = jetpack.read('app/styles/test.less');

    less.render(lessfile)
    .then(function (output) {
        jetpack.file('.tmp/dist/styles/test.css', { content: output.css });
    },
    function (error) {
        console.log(error.toString())
    });

});
```

#### Configuring the Staging Server
``` javascript
task('server', ['copy'], function () {
    connect().use(serveStatic('.tmp/dist')).listen(8080);
    console.log('Listening on port 80.');
});
```
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
#### Minify HTML files with htmlmin
Next we need to compress our html files using htmlmin a simple html 
minifier.
```
npm install --save-dev htmlmin
```
Our jake like following:

``` javascript
	var htmlmin = require('htmlmin');
	....
	task('htmlmin', function() {
		console.log('Minifiying html files....');
		
		var files = new jake.FileList();
		files.include(build + '/*.html');
		
		files.toArray().forEach(function(f) {
			var content = jetpack.read(f);
			jetpack.write(f, htmlmin(content, {
				cssmin: true,
				jsMin: true
			}));
		});
		
	});
	
```

We reads each html file in build dir using jetpack and built in 
jake FileList api. Then we use htmlmin module compress the html 
files. cssMin property says that minify inline css and jsmin says
that minify the inline script. 

#### Minify the css files with CssMin
Cssmin is the simple node.js module that minifies the css files.
``` npm install --save-dev cssmin ```
jake task

``` javascript 
	var cssmin = require('cssmin');
	//...
	task('htmlmin', function() {
		console.log('Minifiying html files....');
		
		var files = new jake.FileList();
		files.include(dist + '/app.css');
		
		files.toArray().forEach(function(f) {
			var content = jetpack.read(f);
			jetpack.write(f, cssmin(content));
		});
		
	});
```
#### Optimize images
``` javascript
task('imagemin', function () {
    new Imagemin()
        .src('images/*.{gif,jpg,png,svg}')
        .dest('build/images')
        .use(Imagemin.jpegtran({ progressive: true }))
    .run(function (err, files) {
        console.log(files[0]);
        // => {path: 'build/images/foo.jpg', contents: <Buffer 89 50 4e ...>} 
    });
});
```

#### Implement the live reload
#### Wrapping up
#### Summary