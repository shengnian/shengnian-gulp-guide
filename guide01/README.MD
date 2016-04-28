Gulp入门实践指南零一
===

# Gulp
----
## 一步 安装
命令行中执行：

`npm install -g gulp` //全局安装

`npm install —-save-dev gulp` //项目本地安装，保存为开发依赖

## 二步 插件
根据项目中需要用到的任务安装不同的插件模块,例:
`npm install —-save-dev gulp-uglify`

## 三步 配置
在项目目录下建立`gulpfile.js`文件,如下:
```
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('minify', function () {
  gulp.src('js/app.js')
    .pipe(uglify())
    .pipe(gulp.dest('build'))
});
```
## 四步 执行
`gulp minify`

 gulp充分使用了“管道”思想，就是一个数据流（stream）：src方法读入文件产生数据流，dest方法将数据流写入文件，中间是一些中间步骤，每一步都对数据流进行一些处理。

## 五步 方法

### 1. src()

gulp模块的src方法，用于产生数据流。它的参数表示所要处理的文件，这些指定的文件会转换成数据流。参数的写法一般有以下几种形式。

```
js/app.js：指定确切的文件名。
js/*.js：某个目录所有后缀名为js的文件。
js/**/*.js：某个目录及其所有子目录中的所有后缀名为js的文件。
!js/app.js：除了js/app.js以外的所有文件。
*.+(js	css)：匹配项目根目录下，所有后缀名为js或css的文件。
```
src方法的参数还可以是一个数组，用来指定多个成员。

`gulp.src(['js/**/*.js', '!js/**/*.min.js'])`

### 2. dest()
dest方法将管道的输出写入文件，同时将这些输出继续输出，所以可以依次调用多次dest方法，将输出写入多个目录。如果有目录不存在，将会被新建。
```
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```
dest方法还可以接受第二个参数，表示配置对象。
```
gulp.dest('build', {
  cwd: './app',
  mode: '0644'
})
```
配置对象有两个字段。cwd字段指定写入路径的基准目录，默认是当前目录；mode字段指定写入文件的权限，默认是0777。

### 3.task()
task方法用于定义具体的任务。它的第一个参数是任务名，第二个参数是任务函数。下面是一个非常简单的任务函数。
```
gulp.task('greet', function () {
   console.log('Hello world!');
});
```

task方法还可以指定按顺序运行的一组任务。
```
gulp.task('build', ['css', 'js', 'imgs']);
```
上面代码先指定build任务，它由css、js、imgs三个任务所组成，task方法会并发执行这三个任务。注意，由于每个任务都是异步调用，所以没有办法保证js任务的开始运行的时间，正是css任务运行结束。

如果希望各个任务严格按次序运行，可以把前一个任务写成后一个任务的依赖模块。
```
gulp.task('css', ['greet'], function () {
   // Deal with CSS here
});
```
上面代码表明，css任务依赖greet任务，所以css一定会在greet运行完成后再运行。

task方法的回调函数，还可以接受一个函数作为参数，这对执行异步任务非常有用。

```
// 执行shell命令
var exec = require('child_process').exec;
gulp.task('jekyll', function(cb) {
  // build Jekyll
  exec('jekyll build', function(err) {
    if (err) return cb(err); // return error
    cb(); // finished task
  });
});
```

如果一个任务的名字为default，就表明它是“默认任务”，在命令行直接输入gulp命令，就会运行该任务。

```
gulp.task('default', function () {
  // Your default task
});

// 或者

gulp.task('default', ['styles', 'jshint', 'watch']);
```
执行的时候，直接使用gulp，就会运行styles、jshint、watch三个任务。

### 4.watch()
watch方法用于指定需要监视的文件。一旦这些文件发生变动，就运行指定任务。
```
gulp.task('watch', function () {
   gulp.watch('templates/*.tmpl.html', ['build']);
});
```
上面代码指定，一旦templates目录中的模板文件发生变化，就运行build任务。

watch方法也可以用回调函数，代替指定的任务。
```
gulp.watch('templates/*.tmpl.html', function (event) {
   console.log('Event type: ' + event.type);
   console.log('Event path: ' + event.path);
});
```
另一种写法是watch方法所监控的文件发生变化时（修改、增加、删除文件），会触发change事件。可以对change事件指定回调函数。
```
var watcher = gulp.watch('templates/*.tmpl.html', ['build']);

watcher.on('change', function (event) {
   console.log('Event type: ' + event.type);
   console.log('Event path: ' + event.path);
});
```
除了change事件，watch方法还可能触发以下事件。
```
end：回调函数运行完毕时触发。
error：发生错误时触发。
ready：当开始监听文件时触发。
nomatch：没有匹配的监听文件时触发。
```
watcher对象还包含其他一些方法。
```
watcher.end()：停止watcher对象，不会再调用任务或回调函数。
watcher.files()：返回watcher对象监视的文件。
watcher.add(glob)：增加所要监视的文件，它还可以附件第二个参数，表示回调函数。
watcher.remove(filepath)：从watcher对象中移走一个监视的文件。
```
## 六步 gulp-load-plugins模块

一般情况下，gulpfile.js中的模块需要一个个加载。
```
var gulp = require('gulp'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    concat = require('gulp-concat');

gulp.task('js', function () {
   return gulp.src('js/*.js')
      .pipe(jshint())
      .pipe(jshint.reporter('default'))
      .pipe(uglify())
      .pipe(concat('app.js'))
      .pipe(gulp.dest('build'));
});
```
上面代码中，除了gulp模块以外，还加载另外三个模块。

这种一一加载的写法，比较麻烦。使用gulp-load-plugins模块，可以加载package.json文件中所有的gulp模块。上面的代码用gulp-load-plugins模块改写，就是下面这样。
```
var gulp = require('gulp'),
    gulpLoadPlugins = require('gulp-load-plugins'),
    plugins = gulpLoadPlugins();

gulp.task('js', function () {
   return gulp.src('js/*.js')
      .pipe(plugins.jshint())
      .pipe(plugins.jshint.reporter('default'))
      .pipe(plugins.uglify())
      .pipe(plugins.concat('app.js'))
      .pipe(gulp.dest('build'));
});
```
上面代码假设package.json文件包含以下内容。
```

{
   "devDependencies": {
      "gulp-concat": "~2.2.0",
      "gulp-uglify": "~0.2.1",
      "gulp-jshint": "~1.5.1",
      "gulp": "~3.5.6"
   }
}
```

## 七步  BrowserSync
`npm install --save-dev browser-sync`
`gulpfile.js`中
```
var gulp        = require('gulp');
var browserSync = require('browser-sync').create();

// 静态服务器
gulp.task('browser-sync', function() {
    browserSync.init({
        server: {
            baseDir: "./"
        }
    });
});

// 代理

gulp.task('browser-sync', function() {
    browserSync.init({
        proxy: "你的域名或IP"
    });
});
```

###  SASS + CSS 注入
通过流的方式创建任务流程, 这样您就可以在您的任务完成后调用`reload`，所有的浏览器将被告知的变化并实时更新. 因为Browsersync只在乎您的CSS在编译完成 `finished` 后 - 注意： `gulp.dest` 后调用重载。
```
var gulp        = require('gulp');
var browserSync = require('browser-sync').create();
var sass        = require('gulp-sass');
var reload      = browserSync.reload;

// 静态服务器 + 监听 scss/html 文件
gulp.task('serve', ['sass'], function() {

    browserSync.init({
        server: "./app"
    });

    gulp.watch("app/scss/*.scss", ['sass']);
    gulp.watch("app/*.html").on('change', reload);
});

// scss编译后的css将注入到浏览器里实现更新
gulp.task('sass', function() {
    return gulp.src("app/scss/*.scss")
        .pipe(sass())
        .pipe(gulp.dest("app/css"))
        .pipe(reload({stream: true}));
});

gulp.task('default', ['serve']);
```

### SASS & Source Maps

如果您使用 gulp-ruby-sass 与 sourcemap: true 选项, 将会生成.map 文件. 这些文件最终会被送到下游，在 browserSync.reload() 时接收它们, 它会尝试重新加载页面 (因为它不会在DOM里发现在任何 .map 文件)。

要解决这个问题, 您可以使用 gulp-filter 包， 以确保只有 \*.css 文件响应 .reload \- 这样一来，您还是会得到CSS注入，而不是整个页面重载。
```
var gulp        = require("gulp");
var sass        = require("gulp-ruby-sass");
var filter      = require('gulp-filter');
var browserSync = require("browser-sync").create();

// 静态服务器 \+ 监听 scss/html 文件
gulp.task('serve', ['sass'], function() {

    browserSync.init({
        server: "./app"
    });

    gulp.watch("app/scss/*.scss", ['sass']);
    gulp.watch("app/*.html").on('change', reload);
});

gulp.task('sass', function () {
    return gulp.src('scss/**/*.scss')
        .pipe(sass({sourcemap: true}))
        .pipe(gulp.dest('css'))// Write the CSS & Source maps
        .pipe(filter('**/*.css')) // Filtering stream to only css files
        .pipe(browserSync.reload({stream:true}));
});
```

### 浏览器重载
有时候，您可能只是想完全重新加载页面 (例如，处理一堆JS文件后), 但您希望在任务发生后重载。这将在 gulp 4.x.x 时变得更容易, 但现在您可以做到以下几点： (确保您 return 从您的工作流 ，以确保 browserSync.reload() 被调用在正确的时间。
```
// 处理完JS文件后返回流
gulp.task('js', function () {
    return gulp.src('js/*js')
        .pipe(browserify())
        .pipe(uglify())
        .pipe(gulp.dest('dist/js'));
});

// 创建一个任务确保JS任务完成之前能够继续响应
// 浏览器重载
gulp.task('js-watch', ['js'], browserSync.reload);

// 使用默认任务启动Browsersync，监听JS文件
gulp.task('serve', ['js'], function () {

    // 从这个项目的根目录启动服务器
    browserSync({
        server: {
            baseDir: "./"
        }
    });

    // 添加 browserSync.reload 到任务队列里
    // 所有的浏览器重载后任务完成。
    gulp.watch("js/*.js", ['js-watch']);
});
```

### 手动重载
如果流支持并不满足您的需求，您可以通过创建一个任务来手动触发重载方法。我们创建了一个sass任务,在我们修改文件后会在浏览器里注入CSS，但是当HTML文件被更改时，浏览器将会被重新加载（手动重载）。
```
// 使用变量引用 `reload` 方法
var reload = browserSync.reload;

// 编译 SASS & 自动注入到浏览器
gulp.task('sass', function () {
    return gulp.src('scss/styles.scss')
        .pipe(sass({includePaths: ['scss']}))
        .pipe(gulp.dest('css'))
        .pipe(reload({stream:true}));
});

// 监听scss和html文件, 当文件发生变化后做些什么！
gulp.task('serve', ['sass'], function () {

    // 从这个项目的根目录启动服务器
    browserSync({
        server: {
            baseDir: "./"
        }
    });

    gulp.watch("scss/*.scss", ['sass']);
    gulp.watch("*.html").on("change", browserSync.reload);
});
```

## 七步 gulp-livereload模块
gulp-livereload模块用于自动刷新浏览器，反映出源码的最新变化。它除了模块以外，还需要在浏览器中安装插件，用来配合源码变化。
```
var gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload'),
    watch = require('gulp-watch');

gulp.task('less', function() {
   gulp.src('less/*.less')
      .pipe(watch())
      .pipe(less())
      .pipe(gulp.dest('css'))
      .pipe(livereload());
});
```
上面代码监视less文件，一旦编译完成，就自动刷新浏览器。

## 参考
[Building With Gulp](https://www.smashingmagazine.com/2014/06/building-with-gulp/)
