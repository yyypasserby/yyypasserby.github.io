---
title: Practices and Skills from NPM to Angular
date: 2015-06-03 11:22:33
categories:
  - web
  - frontend
tags:
  - npm
  - yeoman
  - grunt
  - bower
  - angular
---

![AngularJS](/images/npm-yo-grunt-bower-angular/angular.jpg)

Recently, I am refactorying some of my projects. It may be more proper to say that the projects were rewritten because I have update the technical solutions that is much cooler.

In this blog, I am going to talk about some obstacles I met in building an [Angular](https://angularjs.org/) App and how I solve them. This will not contain real coding technics in how to write an app, but some useful tools or components that may help to write Angular more comfortably and conveniently.

Ok, let's just begin with how to design the project structure of Angular.

Angular can be used in many ways, it provides a lot of useful functions and directives for you to build a web application, which also means Angular is somewhat heavier than other framework. It mainly solves some key problems in the web front-end development, like modules, customizing tags and etc., and assembles all these solutions into a big framework. Consequently, you can use Angular to render your web page and use controllers to bind your data, just like the old ways, or you may define your own directives and services to build an web app in a new way. If you are the second-way supporter, I'm going to talk about a good way to structure your apps.

The main tools we will use next: npm, yo, grunt, bower, angular-generator.

## npm

[npm](https://github.com/npm/npm) is an amazing tool for you if your project is related to `Javascript`. With the development of `Javascript`, npm is also becoming more widely used in development. But this time, it is just a tool installer, we will just use it to prepare the tool for our app.

```bash
npm install -g grunt-cli bower yo generator-karma generator-angular
```

`-g` means that you are going to install these package in a global scope. You can also remove this option, however, I really recommend you to add it because these are all tools which we will use next, even in the future when you need to build an Angular App and they will not contaminate the namespace between the projects.

If you decide to install it in a global scope, it is helpful to customize the directory to store the global resources.

```bash
npm set config prefix path/to/your/directory/[node-global]
npm set config cache path/to/your/directory/[node-cache]
```

`[]` means you can do in that way to organize your npm global directory and I think it is elegant. You could also find the configuration file `~/.npmrc` and define these in it.

The installation process may be very slow in China. You could use an [npm mirror](http://npm.taobao.org/) maintained by Taobao to improve the speed.

```bash
registry=https://registry.npm.taobao.org/
```

## yo & generator-karma & generator-angular

[Yeoman](http://yeoman.io/) is a tool to give you an easy way to build a basic structure for the application. It predefines some kinds of structure of a particular app. [generator-karma](https://github.com/yeoman/generator-karma) and [generator-angular](https://github.com/yeoman/generator-angular) are those predefined structures and they are also need to be installed.

[Karma](http://karma-runner.github.io/0.12/index.html) is a JS unit-test framework. You can define some test in the karma.conf.js and run `karma start karma.conf.js` to make a automatic test for your app. It is especiallly useful when your project is based on some big frameworks like Angular, because it can test every small module you define like a directve, a filter or a controller, and you can integrate your web app continuously and errorlessly. Also it can be registered in a **grunt task** which can be executed automatically once you want to commit a version.

These two generator not only create a directory structure for each of their frameworks. It also defines what needs to be involved in this project. 

Let's use Angular as an example. When you finishes installing the yo and angular-generator, you can now set out to build an Angular App.

```bash
mkdir angular-app
cd angular-app
yo angular
```

After, you can set some setting to your angular project. Yo will automactically download the dependencies by `bower` and inject them in your project using wiredep by `grunt`. Also some framework like `karma` mentioned above will be installed.

yo & angular-generator also do other awesomethings. It will organize the project even you are going to add something to it, like if you are going to add an filter to this app.

```bash
yo angular:filter your_filter_name
```

Yo will create a directory in your `scripts` directory and create a file named `your_filter_name.js` in it. This filter will also be inject into your `index.html` automatically using `wiredep`. You have to pay attention to your filter name, `-` based and customized prefix is recommended. For example,

```bash
yo angular:directive my-quiz-directive
```

A `my-quiz-directive.js` file will be created and a `myQuizDirective` directive will be added to Angular. So `my-quiz-directive` can be used in html like `<my-quiz-directive></my-quiz-directive>` or other Angular recommended ways.

## bower

There is not much to talk about [bower](http://bower.io/). It is first introduced in this [blog]({filename}/2014-12-18-django-and-bower.md). In the Angular app generated by yo, only one thing need to be carefully maintained, `bower.json`. If you want to use the `wiredep` to automate your project dependency, every time your installed a library or a framework by bower, you must use `--save` option to update the `bower.json` file, and the grunt will watch the change of `bower.json` and inject all the dependency in it to the `index.html`

## grunt

[grunt](http://gruntjs.com/) is an automator in web front-end development.

### Why we need an automator?
After several years of exploration in web dev, engineers find many best practices in web front-end dev, including some effort to do with performance and some with engineering. And all these tasks like test, wiredep, cssmin, htmlmin, uglify, all can be done automatically. So we just need a way to define how it works and then let a program does it for us.

grunt is that kind of thing. Also [gulp](http://gulpjs.com/) can do the same thing, but lighter.

You can define the tasks in `Gruntfile.js`, most of the tasks you need is already predefined in grunt, so you just need to call them by name.

Talking about our app, the basic tasks generated automactically by yo are following in my space:

- clean
- wiredep
- useminPrepare
- concurrent
- autoprefixer
- concat
- ngAnnotate
- copy
- cdnify
- cssmin
- uglify
- filerev
- usemin
- htmlmin
- karma
- jshint
- watch

You can reorganize these module into tasks and call them.

```bash
grunt task_name
```

Several changes I made to the default `Gruntfile.js`:

#### Add ignore symbols to jshint
[jshint](http://jshint.com/) is to check the correctness of your js code. However, you may find some warnings when you run jshint, this may abort the whole grunt tasks if you do not add `--force` options. You can modify `.jshintrc` file in the current project directory and add your symbols ignored by the jshint.

```json
{
    "globals": {
        "angular": false,
        "$": false
    }
}
```

This means `angular` and `$` symbols will be ignored when checking the js syntax.

#### Add css-dependent resources copied
When you use some front-end framework like [bootstrap](http://getbootstrap.com/), you may encounter some problems like web page cannot locate the image resources. That is because in the production environment the theme will be copied, only css and js. So you must modify the gruntfile.

This action is provided by the `copy` module in `grunt`.

```json
{
    expand: true,
    dot: true,
    cwd: 'path/to/your/cwd',
    dest: 'path/to/your/dest',
    src: ['paths/to/your/src']
}
```

You can add above structure to the `copy:dist:files` in `Gruntfile.js`, add the all the srcs your defined in `path/to/your/cwd/paths/to/your/src` will be copied to `path/to/your/dest/paths/to/your/src`.

#### Remove dynamic image path from Filerev
Filerev is to renames files for browser caching purposes. However, if you have some image path is dynamically rendered by the attribute defined by user, it should be excluded.

This function is provided by `filerev` module in grunt.

```json
filerev: {
    dist: {
    src: [
        '<%= yeoman.dist %>/scripts/{,*/}*.js',
        '<%= yeoman.dist %>/styles/{,*/}*.css',
        '<%= yeoman.dist %>/images/{,*/}*.{png,jpg,jpeg,gif,webp,svg}',
        '<%= yeoman.dist %>/styles/fonts/*'
    ]
    }
}
```

You need to remove the src from it or redefine the static resources path for browser caching purposes.

#### Test modification
If you use the yo way to create directives, your test case is also generated in the `test` directory. Consequently, you should write your own test cases, not `awesomeThings` any more. :-)

Also some modules which need not to be tested should be remove in `karma.conf.js`.

## Conclusion

With all the tools, to me, I feel like I have a commandline-based IDE. Also if you preferred a GUI-way to develop, you can choose many other **real** IDE, like [Webstorm](https://www.jetbrains.com/webstorm/).
