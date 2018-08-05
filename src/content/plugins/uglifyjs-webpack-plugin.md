---
title: UglifyjsWebpackPlugin
source: https://raw.githubusercontent.com/webpack-contrib/uglifyjs-webpack-plugin/master/README.md
edit: https://github.com/webpack-contrib/uglifyjs-webpack-plugin/edit/master/README.md
repo: https://github.com/webpack-contrib/uglifyjs-webpack-plugin
---
This plugin uses <a href="https://github.com/mishoo/UglifyJS2/tree/harmony">UglifyJS v3 </a><a href="https://npmjs.com/package/uglify-es">(`uglify-es`)</a> to minify your JavaScript

> ℹ️  `webpack < v4.0.0` currently contains [`v0.4.6`](https://github.com/webpack-contrib/uglifyjs-webpack-plugin/tree/version-0.4) of this plugin under `webpack.optimize.UglifyJsPlugin` as an alias. For usage of the latest version (`v1.0.0`), please follow the instructions below. Aliasing `v1.0.0` as `webpack.optimize.UglifyJsPlugin` is scheduled for `webpack v4.0.0`

## 安装

```bash
npm i -D uglifyjs-webpack-plugin
```

## 用法

**webpack.config.js**
```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  //...
  optimization: {
    minimizer: [
      new UglifyJsPlugin()
    ]
  }
}
```

## 选项

|属性名称|类型|默认值|描述|
|:--:|:--:|:-----:|:----------|
|**`test`**|`{RegExp\|Array<RegExp>}`| <code>/\.js($&#124;\?)/i</code>|测试匹配的文件|
|**`include`**|`{RegExp\|Array<RegExp>}`|`undefined`|`包含`的文件|
|**`exclude`**|`{RegExp\|Array<RegExp>}`|`undefined`|`排除`的文件。|
|**`cache`**|`{Boolean\|String}`|`false`|启用文件缓存|
|**`cacheKeys`**|`{Function(defaultCacheKeys, file) -> {Object}}`|`defaultCacheKeys => defaultCacheKeys`|Allows you to override default cache keys|
|**`parallel`**|`{Boolean\|Object}`|`false`|使用多进程并行运行和文件缓存来提高构建速度|
|**`sourceMap`**|`{Boolean}`|`false`|使用 source map 将错误信息的位置映射到模块（这会减慢编译的速度） ⚠️ **`cheap-source-map` 选项不适用于此插件**|
|**`minify`**|`{Function}`|`undefined`|Allows you to override default minify function|
|**`uglifyOptions`**|`{Object}`|[`{...defaults}`](https://github.com/webpack-contrib/uglifyjs-webpack-plugin/tree/master#uglifyoptions)|`uglify` [选项](https://github.com/mishoo/UglifyJS2/tree/harmony#minify-options)|
|**`extractComments`**|`{Boolean\|RegExp\|Function<(node, comment) -> {Boolean\|Object}>}`|`false`|是否将注释提取到单独的文件，（查看[详细](https://github.com/webpack/webpack/commit/71933e979e51c533b432658d5e37917f9e71595a(`webpack >= 2.3.0`)|
|**`warningsFilter`**|`{Function(source) -> {Boolean}}`|`() => true`|允许过滤 uglify 警告|

##

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    test: /\.js($|\?)/i
  })
]
```

### `include`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    include: /\/includes/
  })
]
```

### `exclude`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    exclude: /\/excludes/
  })
]
```

### `cache`

If you use your own `minify` function please read the `minify` section for cache invalidation correctly.

#### `{Boolean}`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    cache: true
  })
]
```

Enable file caching.
Default path to cache directory: `node_modules/.cache/uglifyjs-webpack-plugin`.

#### `{String}`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    cache: 'path/to/cache'
  })
]
```

Path to cache directory.

### `cacheKeys`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    cache: true,
    cacheKeys: (defaultCacheKeys, file) => {
      defaultCacheKeys.myCacheKey = 'myCacheKeyValue';

      return defaultCacheKeys;
    },
  })
]
```

Allows you to override default cache keys.

Default keys:
```js
{
  'uglify-es': versions.uglify, // uglify version
  'uglifyjs-webpack-plugin': versions.plugin, // plugin version
  'uglifyjs-webpack-plugin-options': this.options, // plugin options
  path: compiler.outputPath ? `${compiler.outputPath}/${file}` : file, // asset path
  hash: crypto.createHash('md4').update(input).digest('hex'), // source file hash
}
```

### `parallel`

#### `{Boolean}`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    parallel: true
  })
]
```

Enable parallelization.
Default number of concurrent runs: `os.cpus().length - 1`.

#### `{Number}`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    parallel: 4
  })
]
```

Number of concurrent runs.

> ℹ️  Parallelization can speedup your build significantly and is therefore **highly recommended**

### `sourceMap`

If you use your own `minify` function please read the `minify` section for handling source maps correctly.

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    sourceMap: true
  })
]
```

> ⚠️ **`cheap-source-map` options don't work with this plugin**

### `minify`

> ⚠️ **Always use `require` inside `minify` function when `parallel` option enabled**

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
     minify(file, sourceMap) {
       const extractedComments = [];

       // Custom logic for extract comments

       const { error, map, code, warnings } = require('uglify-module') // Or require('./path/to/uglify-module')
         .minify(
           file,
           { /* Your options for minification */ },
         );

       return { error, map, code, warnings, extractedComments };
      }
  })
]
```

By default plugin uses `uglify-es` package.

Examples:

#### `uglify-js`

```bash
npm i -D uglify-js
```

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    // Uncomment lines below for cache invalidation correctly
    // cache: true,
    // cacheKeys(defaultCacheKeys) {
    //   return Object.assign(
    //     {},
    //     defaultCacheKeys,
    //     { 'uglify-js': require('uglify-js/package.json').version },
    //   );
    // },
    minify(file, sourceMap) {
      // https://github.com/mishoo/UglifyJS2#minify-options
      const uglifyJsOptions = { /* your `uglify-js` package options */ };

      if (sourceMap) {
        uglifyJsOptions.sourceMap = {
          content: sourceMap,
        };
      }

      return require('uglify-js').minify(file, uglifyJsOptions);
    }
  })
]
```

#### `terser`

```bash
npm i -D terser
```

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    // Uncomment lines below for cache invalidation correctly
    // cache: true,
    // cacheKeys(defaultCacheKeys) {
    //   return Object.assign(
    //     {},
    //     defaultCacheKeys,
    //     { terser: require('terser/package.json').version },
    //   );
    // },
    minify(file, sourceMap) {
      // https://github.com/fabiosantoscode/terser#minify-options
      const terserOptions = { /* your `terser` package options */ };

      if (sourceMap) {
        terserOption.sourceMap = {
          content: sourceMap,
        };
      }

      return require('terser').minify(file, terserOptions);
    }
  })
]
```

### [`uglifyOptions`](https://github.com/mishoo/UglifyJS2/tree/harmony#minify-options)

|Name|Type|Default|Description|
|:--:|:--:|:-----:|:----------|
|**`ecma`**|`{Number}`|`undefined`|Supported ECMAScript Version (`5`, `6`, `7` or `8`). Affects `parse`, `compress` && `output` options|
|**`warnings`**|`{Boolean}`|`false`|Display Warnings|
|**[`parse`](https://github.com/mishoo/UglifyJS2/tree/harmony#parse-options)**|`{Object}`|`{}`|Additional Parse Options|
|**[`compress`](https://github.com/mishoo/UglifyJS2/tree/harmony#compress-options)**|`{Boolean\|Object}`|`true`|Additional Compress Options|
|**[`mangle`](https://github.com/mishoo/UglifyJS2/tree/harmony#mangle-options)**|`{Boolean\|Object}`|`{inline: false}`|Enable Name Mangling (See [Mangle Properties](https://github.com/mishoo/UglifyJS2/tree/harmony#mangle-properties-options) for advanced setups, use with ⚠️)|
|**[`output`](https://github.com/mishoo/UglifyJS2/tree/harmony#output-options)**|`{Object}`|`{comments: extractComments ? false : /^\**!\|@preserve\|@license\|@cc_on/,}`|Additional Output Options (The defaults are optimized for best compression)|
|**`toplevel`**|`{Boolean}`|`false`|Enable top level variable and function name mangling and to drop unused variables and functions|
|**`nameCache`**|`{Object}`|`null`|Enable cache of mangled variable and property names across multiple invocations|
|**`ie8`**|`{Boolean}`|`false`|Enable IE8 Support|
|**`keep_classnames`**|`{Boolean}`|`undefined`|Enable prevent discarding or mangling of class names|
|**`keep_fnames`**|`{Boolean}`|`false`| Enable prevent discarding or mangling of function names. Useful for code relying on `Function.prototype.name`. If the top level minify option `keep_classnames` is `undefined` it will be overriden with the value of the top level minify option `keep_fnames`|
|**`safari10`**|`{Boolean}`|`false`|Enable work around Safari 10/11 bugs in loop scoping and `await`|

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    uglifyOptions: {
      ecma: 8,
      warnings: false,
      parse: {...options},
      compress: {...options},
      mangle: {
        ...options,
        properties: {
          // mangle property options
        }
      },
      output: {
        comments: false,
        beautify: false,
        ...options
      },
      toplevel: false,
      nameCache: null,
      ie8: false,
      keep_classnames: undefined,
      keep_fnames: false,
      safari10: false,
    }
  })
]
```

### `extractComments`

#### `{Boolean}`

All comments that normally would be preserved by the `comments` option will be moved to a separate file. If the original file is named `foo.js`, then the comments will be stored to `foo.js.LICENSE`.

#### `{RegExp|String}` or  `{Function<(node, comment) -> {Boolean}>}`

All comments that match the given expression (resp. are evaluated to `true` by the function) will be extracted to the separate file. The `comments` option specifies whether the comment will be preserved, i.e. it is possible to preserve some comments (e.g. annotations) while extracting others or even preserving comments that have been extracted.

#### `{Object}`

|Name|Type|Default|Description|
|:--:|:--:|:-----:|:----------|
|**`condition`**|`{Regex\|Function}`|``|通常表达式或者相应函数（见上文）|
|**`filename`**|`{String\|Function}`|`${file}.LICENSE`|提取注释的文件会被存储。可以使一个 `{String}` 字符串或者是一个 `{Function<(string) -> {String}>}` 返回字符串的函数，作为原文件名。默认加上文件后缀名 `.LICENSE`|
|**`banner`**|`{Boolean\|String\|Function}`|`/*! For license information please see ${filename}.js.LICENSE */`|banner 文本会在原文件的头部指出被提取的文件，会在源文件加入该信息。可以是 `false`（表示没有 banner），一个 `{String}`，或者一个 `{Function<(string) -> {String}` 返回字符串的函数，会在提取已经被存储注释的时候被调用。注释会被覆盖。|

### `warningsFilter`

**webpack.config.js**
```js
[
  new UglifyJsPlugin({
    warningsFilter: (src) => true
  })
]
```

## 维护人员

<table>
  <tbody>
    <tr>
      <td align="center">
        <a href="https://github.com/hulkish">
          <img width="150" height="150" src="https://github.com/hulkish.png?size=150">
          </br>
          Steven Hargrove
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/bebraw">
          <img width="150" height="150" src="https://github.com/bebraw.png?v=3&s=150">
          </br>
          Juho Vepsäläinen
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/d3viant0ne">
          <img width="150" height="150" src="https://github.com/d3viant0ne.png?v=3&s=150">
          </br>
          Joshua Wiens
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/michael-ciniawsky">
          <img width="150" height="150" src="https://github.com/michael-ciniawsky.png?v=3&s=150">
          </br>
          Michael Ciniawsky
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/evilebottnawi">
          <img width="150" height="150" src="https://github.com/evilebottnawi.png?v=3&s=150">
          </br>
          Alexander Krasnoyarov
        </a>
      </td>
    </tr>
  <tbody>
</table>


[npm]: https://img.shields.io/npm/v/uglifyjs-webpack-plugin.svg
[npm-url]: https://npmjs.com/package/uglifyjs-webpack-plugin

[node]: https://img.shields.io/node/v/uglifyjs-webpack-plugin.svg
[node-url]: https://nodejs.org

[deps]: https://david-dm.org/webpack-contrib/uglifyjs-webpack-plugin.svg
[deps-url]: https://david-dm.org/webpack-contrib/uglifyjs-webpack-plugin

[test]: https://img.shields.io/circleci/project/github/webpack-contrib/uglifyjs-webpack-plugin.svg
[test-url]: https://circleci.com/gh/webpack-contrib/uglifyjs-webpack-plugin

[cover]: https://codecov.io/gh/webpack-contrib/uglifyjs-webpack-plugin/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/webpack-contrib/uglifyjs-webpack-plugin

[chat]: https://img.shields.io/badge/gitter-webpack%2Fwebpack-brightgreen.svg
[chat-url]: https://gitter.im/webpack/webpack
