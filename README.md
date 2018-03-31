# Gulp-

```javascript

const gulp = require('gulp');
const sass = require('gulp-sass');
const autoprefixer = require('gulp-autoprefixer');
const cssnano = require('gulp-cssnano');
const concat = require('gulp-concat');
const clean = require('gulp-clean');
const notify = require('gulp-notify');
const plumber = require('gulp-plumber');
const sourcemaps   = require('gulp-sourcemaps');
const imagemin = require('gulp-imagemin');
const pngquant = require('imagemin-pngquant');
const browserSync  = require('browser-sync').create();

// Все необходимые нам пути.
const path = {
	dist: {   //Путь куда складывать готовые после сборки файлы.
		html: 'dist/',
    js: 'dist/js/',
    css: 'dist/css',
    img: 'dist/img/', 
   	fonts: 'dist/fonts/'
	},
	app: {   //Пути откуда брать исходники
    html: 'app/**/*.html', 
    js: 'app/js/**/*.js',
    css: 'app/css/*.{scss,sass}', // main css
    img: 'app/img/**/*.*', 
    fonts: 'app/fonts/**/*.*'
    },
  watch: {   //За изменением каких файлов мы хотим наблюдать
    html: 'app/**/*.html',
    js: 'app/js/**/*.js',
    css: 'app/css/**/*.{scss,sass}',
    img: 'app/img/**/*.*',
    fonts: 'app/fonts/**/*.*'
    },
  clean: 'dist/'
};

// Эта конструкция работает синхронно, сначала выполняется задача 'clean' и только после ее завершнения запускается 'dev'.
gulp.task('default', ['clean'], function() {
	gulp.run('dev');
});

// Выполняет задача 'clean' и после ее завершения запускается 'dist'.
gulp.task('production', ['clean'], function() {
	gulp.run('dist');
});

// Задача 'dev' представляется собой сборку в режиме разработки.
// Запускает dist - сборку, watcher - слежку за файлами и browser-sync.
gulp.task('dev', ['dist', 'watch', 'browser-sync']);

// Задача 'dist' представляет собой сборку в режиме продакшен.
// Собирает проект.
gulp.task('dist', ['html', 'styles', 'scripts', 'img', 'fonts']);

// Задача для удаление папки dist.
gulp.task('clean', function() {
	return gulp.src(path.clean)
		.pipe(clean());
});

// Задача 'watch' следит за всеми нашими файлами в проекте и при изменении тех или иных перезапустает соответсвующую задачу.
gulp.task('watch', function() {
	gulp.watch(path.watch.css, ['styles']); 
    gulp.watch(path.watch.js, ['scripts']);
    gulp.watch(path.watch.html, ['html']); 
    gulp.watch(path.watch.img, ['img']);   
    gulp.watch(path.watch.fonts, ['fonts']);
    gulp.watch('app/**/*.*').on('change', browserSync.reload);   //Перезапуск browserSynс.1
});

// Задача 'html' перемещает html-файлы.
gulp.task('html', function() {
	return gulp.src(path.app.html)
		.pipe(gulp.dest(path.dist.html));
})

// Задача 'styles' выполняет сборку наших стилей.
gulp.task('styles', function() {
	return gulp.src(path.app.css)
		.pipe(plumber({   // plumber - плагин для отловли ошибок.
			errorHandler: notify.onError(function(err) {   // nofity - представление ошибок в удобном для вас виде.
				return {
					title: 'Styles',
					message: err.message
				}
			})
		}))
		.pipe(sourcemaps.init())   //История изменения стилей, которая помогает нам при отладке в devTools.
		.pipe(sass({	  //Компиляция sass.
 			outputStyle: 'expanded',
			includePaths: ['./css']})) 
		.pipe(autoprefixer({   //Добавление autoprefixer.
			browsers: ['last 2 versions']
		}))
		.pipe(cssnano())   //Минификация стилей.
		.pipe(sourcemaps.write())
		.pipe(gulp.dest(path.dist.css));
});

// Выполняем сборку скриптов.
gulp.task('scripts', function() {
	return gulp.src(path.app.js)
		.pipe(plumber({   // plumber - плагин для отловли ошибок.
			errorHandler: notify.onError(function(err) {   // nofity - представление ошибок в удобном для вас виде.
				return {
					title: 'Script',
					message: err.message
				}
			})
		}))
		.pipe(concat('scripts.min.js'))  //Собираем файлы.
		.pipe(uglify())   //Минификация скриптов.
    .pipe(gulp.dest(path.dist.js));
});

// Задача для запуска сервера.
gulp.task('browser-sync', function() {
	return browserSync.init({
		server: {
			baseDir: 'dist/',
			directory: true
		}
	});
});

// Оптимизация изоброжений.
gulp.task('img', function() {
	return gulp.src(path.app.img)
		.pipe(imagemin({  // Сжимаем их с наилучшими настройками с учетом кеширования
	      interlaced: true,
	      progressive: true,
	      svgoPlugins: [{removeViewBox: false}],
	      use: [pngquant()]
	  }))
		.pipe(gulp.dest(path.dist.img));
});

// Перемешение наших шрифты в папку dist.
gulp.task('fonts', function() {
	return gulp.src(path.app.fonts)
		.pipe(gulp.dest(path.dist.fonts));
});

 ```
