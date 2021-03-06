## 启动
```bash
#下载
git clone https://github.com/raoenhui/gulpExample.git 

#安装
cd gulpExample && npm i

#开发
npm run dev 

#打开网页
npm run server 

#打包
npm run build 
```
> gulp 3 与 gulp 4 写法不同，本仓库的技术栈是gulp 4
> node版本最好在v8.11.1以上

## 一、前言
有些简单前端小项目，不需要涉及框架，前端打包压缩的话本妹子还是喜欢用`gulp`。

## 二、解决浏览器缓存问题
#### gulp-rev
1.为静态文件添加唯一`hash`值，如 unicorn.css → unicorn-d41d8cd98f.css。
2.生成`map`映射文件，方便后面`html`更换文件名
```javascript
gulp.task('js', () =>
    gulp.src(['./src/app.js', './src/app2.js'])
        .pipe(gulp.dest('dist')) // 将源文件拷贝到打包目录
        .pipe(rev())  
        .pipe(gulp.dest('dist')) // 将生成的hash文件添加到打包目录
        .pipe(rev.manifest('js-rev.json'))
        .pipe(gulp.dest('dist')) // 将map映射文件添加到打包目录
);

gulp.task('css',()=> {
    gulp.src('./src/*.css')
        .pipe(gulp.dest('dist')) // 将生成的hash文件添加到打包目录
        .pipe(rev())
        .pipe(gulp.dest('dist'))// write rev'd assets to build dir
        .pipe(rev.manifest('css-rev.json'))
        .pipe(gulp.dest('dist'))   // 将map映射文件添加到打包目录

});
```
#### gulp-rev-rewrite
根据`rev`生成的manifest.json `map`映射文件, 去替换`html`文件中的引用名称, 
```javascript
gulp.task('html', () => {
  const jsManifest = gulp.src('dist/js-rev.json'); //获取js映射文件
  const cssManifest = gulp.src('dist/css-rev.json'); //获取css映射文件
  return gulp.src('./*.html')
    .pipe(revRewrite({manifest: jsManifest})) // 把引用的js替换成有版本号的名字
    .pipe(revRewrite({manifest: cssManifest})) // 把引用的css替换成有版本号的名字
    .pipe(gulp.dest('dist'))
});
```
替换成功
![image.png](https://upload-images.jianshu.io/upload_images/9902136-e644255241b5f9ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 三、gulp其他常用插件
### JS相关
#### gulp-babel 
`babel`是一个 JavaScript 编译器。我们主要是用将`ES6`转换成可以在浏览器中运行的代码。而`gulp-babel `的用法、功能和`babel `是一样的。
先运行 npm install --save-dev gulp-babel @babel/core @babel/preset-env  @babel/plugin-transform-runtime，装好`babel`。
```javascript
const babel = require('gulp-babel');
gulp.task('js', () =>
    gulp.src('src/app.js')
        .pipe(babel({
            presets: ['@babel/env'], 
            plugins: ['@babel/transform-runtime']
        }))
        .pipe(gulp.dest('dist'))
);
```
#### gulp-sourcemaps 
找到编译源文件，方便调试源码。
```javascript
const sourcemaps = require('gulp-sourcemaps');
gulp.task('js', () =>
    gulp.src('src/app.js')
    .pipe(sourcemaps.init())
        .pipe(babel({
            presets: ['@babel/env'], 
            plugins: ['@babel/transform-runtime']
        }))
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest('dist'))
);
```
![image.png](https://upload-images.jianshu.io/upload_images/9902136-3477f46bd1901a19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### gulp-concat
合并`js`文件
```javascript 
const concat = require('gulp-concat');
gulp.task('js', function() {
  return gulp.src(['./src/app.js', './src/app2.js'])
    .pipe(concat('app.js'))
    .pipe(gulp.dest('dist'));
});             
```
#### gulp-uglify
压缩`js`文件
```javascript 
const uglify= require('gulp-uglify');
gulp.task('js', function() {
  return gulp.src(['./src/app.js', './src/app2.js'])
    .pipe(uglify())
    .pipe(gulp.dest('dist'));
});             
```

### CSS相关
#### gulp-postcss
`CSS`预处理器。
```javascript
const postcss = require('gulp-postcss');
const autoprefixer = require('autoprefixer'); //添加css兼容性写法
gulp.task('css', function () {
    return gulp.src('./src/*.css')
        .pipe(postcss([ autoprefixer({
            browsers: [
                '>1%',
                'last 4 versions',
                'Firefox ESR',
                'not ie < 9', 
                'iOS >= 8',
                'Android > 4.4'
            ],
            flexbox: 'no-2009',
        }) ]))
        .pipe(gulp.dest('./dest'));
});
```
#### gulp-clean-css
压缩`CSS`
```javascript
const cleanCSS = require('gulp-clean-css');
gulp.task('css', () => {
  return gulp.src('styles/*.css')
    .pipe(cleanCSS({compatibility: 'ie8'}))
    .pipe(gulp.dest('dist'));
});
```
### HTML相关
#### gulp-inline-source
将引用的`js`、`css`文件，插入`html`中，变成内联式引用。
```javascript
const inlinesource = require('gulp-inline-source');
gulp.task('html', function () {
    return gulp.src('./*.html')
        .pipe(inlinesource({
           compress: false     //是否压缩成一行，默认为true压缩
         }))
        .pipe(gulp.dest('./out'));
});
```
#### gulp-htmlmin
压缩html
```javascript          
const htmlmin = require('gulp-htmlmin');
gulp.task('minify', () => {
  return gulp.src('src/*.html')
    .pipe(htmlmin({
                removeComments: true,  //去除备注
                collapseWhitespace: true //去除空白
              }))
    .pipe(gulp.dest('dist'));
});    
```

### 其他
#### del 
删除文件或文件夹
```javascript
const del = require('del');
/* 清理一些不是必须的js，css文件 */
gulp.task('clean', function() {
    return del(['./dist/*.js',
        './dist/*.css'
    ]).then(function() {
        console.log('delete unnecessary files for firecrackers');
    });
});
```
#### gulp-rename
重命名文件
```javascript
const rename = require('gulp-rename');
gulp.task('html', function() {
.pipe(rename({
    dirname: ".",                // 路径名
    basename: "index",            // 主文件名
    prefix: "pre-",                 // 前缀
    suffix: "-min",                 // 后缀
    extname: ".html"                // 扩展名
  }))
.pipe(gulp.dest('dist'))
});
```


## 其他链接
> * 原文地址：[https://raoenhui.github.io/js/2019/03/03/gulp](https://raoenhui.github.io/js/2019/03/03/gulp)

Happy coding .. :)
