<p align="center"><image src="https://github.com/classicemi/bowl.js/blob/develop/assets/logo.png?raw=true" width="128"></p>

# bowl.js
[![npm](https://img.shields.io/npm/v/bowl.js.svg?style=flat-square)](https://www.npmjs.com/package/bowl.js)
[![npm](https://img.shields.io/npm/dt/bowl.js.svg?style=flat-square)](https://www.npmjs.com/package/bowl.js)
[![license](https://img.shields.io/github/license/elemefe/bowl.svg?style=flat-square)](https://github.com/ElemeFE/bowl)
[![GitHub stars](https://img.shields.io/github/stars/elemefe/bowl.svg?style=social&label=Star)](https://github.com/ElemeFE/bowl)

**bowl** 是一个用 localStorage 来缓存脚本和样式资源的加载器。在获取脚本和样式之后，这个小巧的 JavaScript 库会将它们保存到浏览器的 localStorage 中。当这个文件下次再被请求的时候，bowl 将会从 localStorage 中读取并将它插入到页面中。

## 安装
``` shell
$ npm install bowl.js
```
然后，在页面中插入一个 `script` 标签（推荐在 `head` 标签中插入）：
``` html
<script src="https://unpkg.com/bowl.js/lib/bowl.min.js"></script>
```
or
``` html
<script src="node_modules/bowl.js/lib/bowl.min.js"></script>
```

## 示例
对于那些需要被缓存的资源，你不需要在页面中写入标签。只需要写一些简单的 JS 脚本，bowl 会替你处理接下来的事情。
```html
<script>
var bowl = new Bowl()
bowl.add([
  { url: 'dist/vendor.[hash].js', key: 'vendor' },
  { url: 'dist/app.[hash].js', key: 'app' },
  { url: 'dist/style.[hash].css', key: 'style' }
])
bowl.inject().then(() => {
  // 你的代码
})
</script>
```
**bowl** 会把这些资源加入缓存（目前是 localStorage）。只要资源的 URL 发生了改变，bowl 会更新缓存中的资源。要了解更多 **bowl** 的功能，请参考 API 文档。  
**受同源策略和内部实现机制限制，bowl 只能对非跨域资源进行缓存，跨域资源无法被缓存，但仍然可以使用 bowl 将它们注入页面。**

## 设置开发环境
克隆仓库之后，运行：
```shell
$ yarn install # 是的，推荐使用 yarn。 :)
```
### 常用 NPM 脚本：
```shell
# 监听并自动重新构建 lib/bowl.js 和 lib/bowl.min.js 文件
$ npm run dev

# 监听并自动在 Chrome 中重新执行单元测试
$ npm run test:unit

# 构建所有发布文件
$ npm run build
```

## 项目结构
+ **`assets`**: logo 文件。
+ **`build`**: 包含所有和构建过程相关的配置文件。
+ **`docs`**: 项目主页及文档。
+ **`lib`**: 包含用来发布的文件，执行 `npm run build` 脚本后，这个目录中的文件会被更新。
+ **`test`**: 包含所有的测试，单元测试使用 [Jasmine](http://jasmine.github.io/2.5/introduction) 编写，依赖 [Karma](http://karma-runner.github.io/1.0/index.html) 执行，e2e 测试使用 [Mocha](https://mochajs.org/)。
+ **`src`**: 源代码目录。

## API
**bowl** 将会在全局对象上添加一个 `Bowl` 类，在浏览器环境中是在 window 对象上。Bowl 的实例包含了一些方法供你使用。  
*bowl 的正常运行需要 localStorage 和 Promise 的支持，如果你的项目需要兼容不支持 Promise 的浏览器，你可以使用 Promise 的 Polyfill，在全局对象上添加 Promise 属性供 bowl 识别即可。*

### `bowl.configure`
`bowl.configure(config)`

*config:* 一个包含 bowl 实例自定义设置的对象，支持的配置项有：
+ **timeout**: 获取资源操作的时间限制（单位为毫秒）。

**示例**
```javascript
bowl.configure({
  timeout: 10000
})
```

### `bowl.add`
`bowl.add(resources)`

*resources:* 包含一系列对象的数组，对象支持的属性如下：
+ **url**(必需): *String* 需要被处理的脚本资源 URI 地址。 由于跨域问题的限制，URI 必需和页面同域，如果不同域的话，资源将不会被缓存。你既可以使用绝对路径，也可以省略 origin。
+ **key**(必需): *String* 用来让 **bowl** 识别资源的名字，缺少这个属性的资源将被忽略。
+ **noCache**: *Boolean* 默认为 `false`。当它为 `true` 时，Bowl 将不会缓存这个资源。
+ **dependencies**: *Array* 包含该资源所有依赖资源的 key 的数组。
+ **expireAfter**: *Number* 指定资源自第一次被缓存起经过多久过期，单位为毫秒。
+ **expireWhen**: *Number* 指定资源在哪个时间点过期，单位为毫秒。
**如果同时指定了 `expireAfter` 和 `expireWhen`，则以 `expireWhen` 为准。**

**示例**
```javascript
bowl.add([
  { url: '/vendor.js', key: 'vendor', expireAfter: 30 * 24 * 60 * 60 * 1000 }
  { url: '/main.js', key: 'main', dependencies: ['vendor'] }
])
```

`bowl.add(resource)`

*resource:* 包含单个资源的对象：
``` javascript
bowl.add({ url: '/main.js', key: 'main' })
```

### `bowl.inject`
`bowl.inject()`

这个方法触发对在 `bowl.add()` 方法中添加的资源进行的处理。Bowl 会检查资源是否在 localStorage 中有缓存。如果没有，bowl 将会从服务器获取资源并将它缓存至 localStorage。该方法返回一个 Promise，当资源全部被注入后 Promise 的回调函数会被执行。
```javascript
bowl.inject().then(() => {
  // 你的代码
})
```

### `bowl.remove`
`bowl.remove(resource)`  
*resource:* 这个参数支持两种类型：  
*String:* 表示将要从 bowl 中被移除的资源的 key。

*Object:* 包含以下属性的对象：
+ **url**: 想要从 bowl 中被移除的资源的 url。
+ **key**: 要被移除资源的 key。

`bowl.remove()`  
参数 `resource` 是可选的。当没有提供 `resource` 参数时，bowl 将会从 bowl 实例和缓存中移除所有的资源。

## 开源许可类型
MIT

## FAQ
Q: 浏览器不是有缓存吗，用浏览器缓存不就好了？  
A: 浏览器的缓存并不可靠，用户有可能关闭缓存或手动清除缓存，另外即使有缓存，如果不是通过 Expire 或 Cache-Control 设定强缓存的话，每次还是要向服务器发送请求确认资源是否过期，bowl 可以完全控制这一环节，也不会发送任何请求。

Q: 它和 basket.js 有什么区别？  
A: basket.js 是很好的项目，bowl 也是受到了它的启发。目前和 basket 相比，bowl 还可以不需要额外配置就可以缓存加载 CSS 资源，不支持缓存（如跨域）和不需要缓存的资源也可以使用 bowl 进行普通加载。资源之间存在相互依赖关系的可以通过声明依赖关系来让 bowl 进行分析并按照依赖顺序加载，并不需要类似 require.js 之类的解决方案（实际上 require.js 也要在模块内部声明依赖关系的）。

Q: 未来是否会和 webpack 或 cooking 进行整合？  
A: 这是个好问题，bowl 未来还有很多事情要做，和周边工具进行整合是很重要的环节，不仅是 webpack 和 cooking，团队内部已经做的和正在做的还有不少好的东西，只要是合适的都有可能接入。

Q: 将来 bowl 还会怎么发展？  
A: 可以做的还有很多，但有些是不是要做可能还需要讨论，目前基本确定要做的包括但不限于：配合服务端进行资源差异化更新，降低更新资源时的数据传输量；接入 web worker 或 indexedDB，提供更新的缓存解决方案；支持更多类型资源的缓存，如字体、图片等，etc
