# Setting up TypeScript

With this setup all your `.ts` files in your `app/scripts/ts` directory (create this directory yourself) will be compiled into `app/scripts/main.js`, with sourcemaps and incremental compilation enabled by default.


## Steps

### 1. Install:
[gulp-typescript](https://github.com/ivogabe/gulp-typescript), 
[event-stream](https://github.com/dominictarr/event-stream), 
[gulp-concat](https://github.com/wearefractal/gulp-concat), 
[gulp-sourcemaps](https://github.com/floridoo/gulp-sourcemaps), 
[event-stream](https://github.com/dominictarr/event-stream) plugins

```sh
$ npm install --save-dev gulp-typescript
$ npm install --save-dev event-stream
$ npm install --save-dev gulp-concat
$ npm install --save-dev gulp-sourcemaps
```

### 2. Add `eventStream` as variable in your `gulpfile.js`
```diff
 var gulp = require('gulp');
 var $ = require('gulp-load-plugins')();
 var browserSync = require('browser-sync');
 var reload = browserSync.reload;
+var eventStream = require('event-stream');
```

### 3. Add TypeScript configuration and create a `scripts` task in your `gulpfile.js`

This compiles `main.ts` and everything referenced from `main.ts` into one `app/scripts/main.js` and declaration files into `scripts/ts/definitions`

```js
var tsProject = $.typescript.createProject({
    declarationFiles: true,
    noExternalResolve: true,
    sortOutput: true
});

gulp.task('scripts', function() {
    var tsResult = gulp.src('app/scripts/ts/**/*.ts')
      .pipe($.sourcemaps.init()) // This means sourcemaps will be generated
      .pipe($.typescript(tsProject));

    return eventStream.merge( // Merge the two output streams, so this task is finished when the IO of both operations are done.
        tsResult.dts.pipe(gulp.dest('app/scripts/ts/definitions')),
        tsResult.js.pipe($.concat('main.js')) // You can use other plugins that also support gulp-sourcemaps
            .pipe($.sourcemaps.write()) // Now the sourcemaps are added to the .js file
            .pipe(gulp.dest('app/scripts'))
    );
});
```

### 4. Add `scripts` as a dependency of `html` in your `gulpfile.js`

```js
gulp.task('html', ['styles', 'scripts'], function () {
    ...
```

### 5. Edit your `watch` task in `gulpfile.js`

These changes ensure that (1) generated `.js` files trigger a live reload, and (2) edits to `.ts` files trigger recompilation.

```diff
  gulp.watch([
    'app/*.html',
    '.tmp/styles/**/*.css',
    'app/scripts/**/*.js',
    'app/images/**/*'
  ]).on('change', reload);

+  gulp.watch('app/scripts/ts/**/*.ts', ['scripts']);
   gulp.watch('app/styles/**/*.css', ['styles', reload]);
   gulp.watch('bower.json', ['wiredep', 'fonts', reload]);  
 });
```


## Usage

- Put your `.ts` files in `app/scripts/ts`, and include main.js in your HTML.

- It's fine to have a mixture of `.js` and `.ts` files in your `app/scripts` directory. But note that the TypeScript compiler will always overwrite main.js with the compiled version.
