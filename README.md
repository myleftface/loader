loader [![Build Status](https://secure.travis-ci.org/JacksonTian/loader.png?branch=master)](http://travis-ci.org/JacksonTian/loader) [![Coverage Status](https://coveralls.io/repos/JacksonTian/loader/badge.png)](https://coveralls.io/r/JacksonTian/loader)
==========================

Node静态资源加载器。该模块通过两个步骤配合完成，代码部分根据环境生成标签。上线时，需要调用minify方法进行静态资源的合并和压缩。

# Usage
## Installation

```bash
$ npm install loader
```

## Example
Controller:

```js
res.render(tpl, {
  Loader: require('loader')
});
```
View:
```html
<%- Loader("/assets/scripts/jqueryplugin.js", "/assets/styles/jqueryplugin.css")
  .js("/assets/scripts/lib/jquery.jmodal.js")
  .js("/assets/scripts/lib/jquery.mousewheel.js")
  .js("/assets/scripts/lib/jquery.tagsphere.js")
  .css("/assets/styles/jquery.autocomplate.css")
  .done(assetsMap, prefix, combo) %>
```

### 环境判别
环境判别由`done`方法的第三个参数决定，如果传入combo值，将决定选用线下版本还是线上版本。如果不传入第三个参数，将由环境变量。如下代码实现：

```
process.env.NODE_ENV === 'production'
```
如果不传入combo，需要设置环境，通过以下代码实现：

```
# 生产环境
export NODE_ENV="production"
# 开发环境
export NODE_ENV="dev"
```
可切换进`example`目录运行示例代码：

```
$ npm start
```

### 线上输出
线上模式将会输出合并和压缩后的地址，该地址从Loader构造参数中得到。

```html
<script src="/assets/scripts/jqueryplugin.md5_hash.js"></script>
<link rel="stylesheet" href="/assets/styles/jqueryplugin.md5_hash.css" media="all" />
```

如果你有CDN地址，可以传入prefix参数，使得可以一键切换到CDN地址上，实现网络加速。以下为结果示例：

```html
<script src="http://cnodejs.qiniudn.com/assets/scripts/jqueryplugin.md5_hash.js"></script>
<link rel="stylesheet" href="http://cnodejs.qiniudn.com/assets/styles/jqueryplugin.css" media="all" />
```

### 线下输出
线下模式输出为原始的文件地址。

```html
<script src="/assets/scripts/lib/jquery.jmodal.js"></script>
<script src="/assets/scripts/lib/jquery.mousewheel.js"></script>
<script src="/assets/scripts/lib/jquery.tagsphere.js"></script>
<link rel="stylesheet" href="/assets/styles/jquery.autocomplate.css" media="all" />
```

## 构建
上文没有提及的重要值是`assetsMap`，这个值需要通过构建产生，类似如下格式：

```json
{
  "/assets/index.min.js":"/assets/index.min.ecf8427e.js",
  "/assets/index.min.css":"/assets/index.min.f2fdeab1.css"
}
```

如果需要线上执行，需要该对象的传入。而该对象需要通过以下构建脚本来生成

```
./node_modules/loader/bin/build <views_dir> <output_dir>
```

以上脚本将会遍历视图目录中寻找`Loader().js().css().done()`这样的标记，然后得到合并文件与实际文件的关系。如以上的`assets/index.min.js`文件并不一定需要真正存在，进行扫描构建后，会将相关的`js`文件进行编译和合并为一个文件。
并且根据文件内容进行md5取hash值，最终生成`/assets/index.min.ecf8427e.js`这样的文件。

遍历完目录后，将这些映射关系生成为`assets.json`文件，这个文件位于`<output_dir>`指定的目录下。使用时请正确引入该文件。具体请参见`example`目录下的代码示例。

## 流程
![流程](./figures/flow.png)

## API
请参见[API文档](http://doxmate.cool/JacksonTian/loader/api.html)。

## LESS支持
Loader中支持`.less`文件与普通的`.css`文件没有差别，通过`.css()`加载即可。

```
<%- Loader("/assets/styles/jqueryplugin.min.css")
  .css("/assets/styles/jquery.autocomplate.css")
  .css("/assets/styles/bootstrap.less")
  .done(assetsMap, prefix, combo) %>
```

默认情况下会输出`.less`的原始内容，需要借助`Loader.less(root)`中间来拦截`.less`文件的请求，它将自动将其转换为CSS内容。示例如下：

```
app.use(Loader.less(__dirname)); // Loader.less一定要在静态文件中间件之前，否则.less文件会被静态文件中间件所处理
app.use('/assets', connect.static(__dirname + '/assets', { maxAge: 3600000 * 24 * 365 }));
```

在扫描静态文件、合并压缩方面，没有任何改动。

## Stylus支持
基本同LESS。Loader中支持`.styl`文件与普通的`.css`文件没有差别，通过`.css()`加载即可。

```
<%- Loader("/assets/styles/jqueryplugin.min.css")
  .css("/assets/styles/jquery.autocomplate.css")
  .css("/assets/styles/bootstrap.styl")
  .done(assetsMap, prefix, combo) %>
```

默认情况下会输出styl的原始内容，需要借助`Loader.stylus(root)`中间来拦截`.styl`文件的请求，它将自动将其转换为CSS内容。示例如下：

```
app.use(Loader.stylus(__dirname)); // Loader.stylus一定要在静态文件中间件之前，否则.styl文件会被静态文件中间件所处理
app.use('/assets', connect.static(__dirname + '/assets', { maxAge: 3600000 * 24 * 365 }));
```

在扫描静态文件、合并压缩方面，没有任何改动。

## CoffeeScript支持
基本同LESS。Loader中支持`.coffee`文件与普通的`.js`文件没有差别，通过`.js()`加载即可。

```
<%- Loader("/assets/home.js")
  .js("/assets/home.coffee")
  .done(assetsMap, prefix, combo) %>
```

默认情况下会输出`.coffee`的原始内容，需要借助`Loader.coffee(root)`中间来拦截`.coffee`文件的请求，它将自动将其转换为JS内容。示例如下：

```
app.use(Loader.coffee(__dirname)); // Loader.coffee一定要在静态文件中间件之前，否则.coffee文件会被静态文件中间件所处理
app.use('/assets', connect.static(__dirname + '/assets', { maxAge: 3600000 * 24 * 365 }));
```

在扫描静态文件、合并压缩方面，没有任何改动。

# License
[MIT license](https://github.com/JacksonTian/loader/blob/master/MIT-License)
