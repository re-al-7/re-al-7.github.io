---
title: "Comprimir CSS y JS con GULP"
date: 2020-03-31
tags: ["desarrollo"]
---

En éste articulo vamos a ver como se puede usar GULP para automatizar la reducción de archivos JS y CSS.

![_config.yml](/images/2020-03-31-Comprimir-archivos-con-Gulp.png)

### Gulp

[Gulp](https://gulpjs.com/) es un conjunto de herramientas JavaScript _open-source_, contruido en Node.js y NPM, que ayuda a automatizar tareas comunes en el desarrollo de una aplicación, como pueden ser: mover archivos de una carpeta a otra, eliminarlos, minificar código, etc.

### Minimizar archivos usando Gulp

 1. Se debe tener instalado Node.js, desde el [sitio oficial](https://nodejs.org/). Ademas es necesario instalar las siguientes herramientas:
 
 ~~~powershell
npm i --global cssnano
npm i --global del
npm i --global gulp
npm i --global gulp-concat
npm i --global gulp-cssmin
npm i --global gulp-gzip
npm i --global gulp-header
npm i --global gulp-htmlmin
npm i --global gulp-postcss
npm i --global gulp-rename
npm i --global gulp-uglify
npm i --global gulp-util
npm i --global gulp-zip
npm i --global merge-stream
npm i --global run-sequence
 ~~~

 2. Se debe identificar cuáles y dónde están los archivos JS y CSS a _minimizar_. Esta información la ponemos en un archivo llamado 'bundleconfig.json':

 ~~~js
[
  {
    "outputFileName": "wwwroot/css/integrate.min.css",
    "inputFiles": [
      "wwwroot/lib/bootstrap/dist/css/bootstrap.min.css",
      "wwwroot/lib/font-awesome/css/font-awesome.min.css",
      "wwwroot/lib/bootstrap-datepicker/dist/css/bootstrap-datepicker3.min.css",
      "wwwroot/lib/bootstrap-select/dist/css/bootstrap-select.min.css",
      "wwwroot/css/styles.css"
    ]
  },
  {
    "outputFileName": "wwwroot/js/integrate.min.js",
    "inputFiles": [
      "wwwroot/lib/jquery/dist/jquery.min.js",
      "wwwroot/lib/bootstrap/dist/js/bootstrap.min.js",
      "wwwroot/lib/bootstrap-datepicker/dist/js/bootstrap-datepicker.min.js",
      "wwwroot/lib/bootstrap-select/dist/js/bootstrap-select.min.js",
      "wwwroot/js/custom.js"
    ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    "sourceMap": false
  }
]
 ~~~

 3. Se debe crear el archivo 'gulpfile.js' y añadir el código requerido en la cabecera:

 ~~~js
"use strict";

var gulp = require("gulp"),
    concat = require("gulp-concat"),
    cssmin = require("gulp-cssmin"),
    htmlmin = require("gulp-htmlmin"),
    uglify = require("gulp-uglify"),
    merge = require("merge-stream"),
    del = require("del"),
    gzip = require('gulp-gzip'),
    bundleconfig = require("./bundleconfig.json");

var regex = {
    css: /\.css$/,
    html: /\.(html|htm)$/,
    js: /\.js$/
};
 ~~~

 4. Se procede a definir las posibles tareas. En nuestro ejemplo vamos a definir 8 _tasks_:
 

 ~~~js
gulp.task('gzip_js', () => {
    return gulp.src('wwwroot/js/ziran.min.js')
        .pipe(gzip())
        .pipe(gulp.dest('wwwroot/js/'));
});

gulp.task('gzip_css', () => {
    return gulp.src('wwwroot/css/ziran.min.css')
        .pipe(gzip())
        .pipe(gulp.dest('wwwroot/css/'));
});

gulp.task('min_js', () => {
    var tasks = getBundles(regex.js).map(function (bundle) {
        return gulp.src(bundle.inputFiles, { base: "." })
            .pipe(concat(bundle.outputFileName))
            .pipe(uglify())
            .pipe(gulp.dest("."));
    });
    return merge(tasks);
});

gulp.task('min_css', () => {
    var tasks = getBundles(regex.css).map(function (bundle) {
        return gulp.src(bundle.inputFiles, { base: "." })
            .pipe(concat(bundle.outputFileName))
            .pipe(cssmin())
            .pipe(gulp.dest("."));
    });
    return merge(tasks);
});

gulp.task('min_html', () => {
    var tasks = getBundles(regex.html).map(function (bundle) {
        return gulp.src(bundle.inputFiles, { base: "." })
            .pipe(concat(bundle.outputFileName))
            .pipe(htmlmin({ collapseWhitespace: true, minifyCSS: true, minifyJS: true }))
            .pipe(gulp.dest("."));
    });
    return merge(tasks);
});

gulp.task('clean', () => {
    var files = bundleconfig.map(function (bundle) {
        return bundle.outputFileName;
    });

    return del(files);
});

gulp.task('watch', () => {
    getBundles(regex.js).forEach(function (bundle) {
        gulp.watch(bundle.inputFiles, ["min:js"]);
    });

    getBundles(regex.css).forEach(function (bundle) {
        gulp.watch(bundle.inputFiles, ["min:css"]);
    });

    getBundles(regex.html).forEach(function (bundle) {
        gulp.watch(bundle.inputFiles, ["min:html"]);
    });
});

gulp.task('default', gulp.series(['min_js', 'min_css', 'gzip_js', 'gzip_css']));
 ~~~

Finalmente tenemos una función auxiliar:

~~~js
function getBundles(regexPattern) {
    return bundleconfig.filter(function (bundle) {
        return regexPattern.test(bundle.outputFileName);
    });
}
~~~

Para ejecutar la tarea 'default', se debe ejecutar desde linea de comandos:

~~~powershell
gulp
~~~