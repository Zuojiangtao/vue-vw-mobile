# 如何在Vue项目中使用vw实现移动端适配

有关于移动端的适配布局一直以来都是众说纷纭，对应的解决方案也是有很多种。在《使用Flexible实现手淘H5页面的终端适配》提出了Flexible的布局方案，随着viewport单位越来越受到众多浏览器的支持，因此在《再聊移动端页面的适配》一文中提出了vw来做移动端的适配问题。到目前为止不管是哪一种方案，都还存在一定的缺陷。言外之意，还没有哪一个方案是完美的。

事实上真的不完美？其实不然。最近为了新项目中能更完美的使用vw来做移动端的适配。探讨出一种能解决不兼容viewport单位的方案。今天整理一下，与大家一起分享。如果方案中存在一定的缺陷，欢迎大家一起拍正。

## 准备工作
对于Flexible或者说vw的布局，其原理不在这篇文章进行阐述。如果你想追踪其中的原委，强烈建议你阅读早前整理的文章《使用Flexible实现手淘H5页面的终端适配》和《再聊移动端页面的适配》。

> 说句题外话，由于Flexible的出现，也造成很多同学对rem的误解。正如当年大家对div的误解一样。也因此，大家都觉得rem是万能的，他能直接解决移动端的适配问题。事实并不是如此，至于为什么，我想大家应该去阅读flexible.js源码，我相信你会明白其中的原委。

回到我们今天要聊的主题，怎么实现vw的兼容问题。为了解决这个兼容问题，我将借助Vue官网提供的构建工程以及一些PostCSS插件来完成。在继续后面的内容之前，需要准备一些东西：

* NodeJs
* NPM
* Webpack
* Vue-cli
* postcss-import
* postcss-url
* postcss-aspect-ratio-mini
* postcss-cssnext
* autoprefixer
* postcss-px-to-viewport
* postcss-write-svg
* cssnano
* postcss-viewport-units
* Viewport Units Buggyfill

对于这些起什么作用，先不阐述，后续我们会聊到上述的一些东西。

使用Vue-cli来构建项目
对于NodeJs、NPM和Webpack相关介绍，大家可以查阅其对应的官网。这里默认你的系统环境已经安装好Nodejs、NPM和Webpack。我的系统目前使用的Node版本是v9.4.0；NPM的版本是v5.6.0。事实上，这些都并不重要。

## 使用Vue-cli构建项目

略,自己看官网

## 安装PostCSS插件
通过Vue-cli构建的项目，在项目的根目录下有一个.postcssrc.js，默认情况下已经有了：

```
module.exports = {
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    "autoprefixer": {}
  }
}
```

对应我们开头列的的PostCSS插件清单，现在已经具备了：

* postcss-import
* postcss-url
* autoprefixer

简单的说一下这几个插件。

### postcss-import
`postcss-import`相关配置可以[点击这里](https://github.com/postcss/postcss-import)。目前使用的是默认配置。只在.postcssrc.js文件中引入了该插件。

`postcss-import`主要功有是解决@import引入路径问题。使用这个插件，可以让你很轻易的使用本地文件、node_modules或者web_modules的文件。这个插件配合postcss-url让你引入文件变得更轻松。

### postcss-url
postcss-url相关配置可以点击这里。该插件主要用来处理文件，比如图片文件、字体文件等引用路径的处理。

在Vue项目中，vue-loader已具有类似的功能，只需要配置中将vue-loader配置进去。

### autoprefixer
autoprefixer插件是用来自动处理浏览器前缀的一个插件。如果你配置了postcss-cssnext，其中就已具备了autoprefixer的功能。在配置的时候，未显示的配置相关参数的话，表示使用的是Browserslist指定的列表参数，你也可以像这样来指定last 2 versions 或者 > 5%。

如此一来，你在编码时不再需要考虑任何浏览器前缀的问题，可以专心撸码。这也是PostCSS最常用的一个插件之一。

### 其他插件
Vue-cli默认配置了上述三个PostCSS插件，但我们要完成vw的布局兼容方案，或者说让我们能更专心的撸码，还需要配置下面的几个PostCSS插件：

* postcss-aspect-ratio-mini
* postcss-px-to-viewport
* postcss-write-svg
* postcss-cssnext
* cssnano
* postcss-viewport-units

要使用这几个插件，先要进行安装：

`npm i postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano --S
`

安装成功之后，在项目根目录下的`package.json`文件中，可以看到新安装的依赖包：

```
"dependencies": {
  "cssnano": "^3.10.0",
  "postcss-aspect-ratio-mini": "0.0.2",
  "postcss-cssnext": "^3.1.0",
  "postcss-px-to-viewport": "0.0.3",
  "postcss-viewport-units": "^0.1.3",
  "postcss-write-svg": "^3.0.1",
  "vue": "^2.5.2",
  "vue-router": "^3.0.1"
},
```

接下来在.postcssrc.js文件对新安装的PostCSS插件进行配置：

```
module.exports = {
  "plugins": {
      "postcss-import": {},
      "postcss-url": {},
      "postcss-aspect-ratio-mini": {},
      "postcss-write-svg": {
        utf8: false
      },
    "postcss-cssnext": {},
    "postcss-px-to-viewport": {
      viewportWidth: 750,     // (Number) The width of the viewport.
      viewportHeight: 1334,    // (Number) The height of the viewport.
      unitPrecision: 3,       // (Number) The decimal numbers to allow the REM units to grow to.
      viewportUnit: 'vw',     // (String) Expected units.
      selectorBlackList: ['.ignore', '.hairlines'],  // (Array) The selectors to ignore and leave as px.
      minPixelValue: 1,       // (Number) Set the minimum pixel value to replace.
      mediaQuery: false       // (Boolean) Allow px to be converted in media queries.
    },
    "postcss-viewport-units":{},
    "cssnano": {
      preset: "advanced",
      autoprefixer: false,
      "postcss-zindex": false
    }
  }
}
```

> 特别声明：由于cssnext和cssnano都具有autoprefixer,事实上只需要一个，所以把默认的autoprefixer删除掉，然后把cssnano中的autoprefixer设置为false。对于其他的插件使用，稍后会简单的介绍。

由于配置文件修改了，所以重新跑一下`npm run dev`。项目就可以正常看到了。接下来简单的介绍一下后面安装的几个插件的作用。

### postcss-cssnext
postcss-cssnext其实就是cssnext。该插件可以让我们使用CSS未来的特性，其会对这些特性做相关的兼容性处理。其包含的特性主要有：

postcss-cssnext
有关于cssnext的每个特性的操作文档，可以点击这里浏览。

### cssnano
cssnano主要用来压缩和清理CSS代码。在Webpack中，cssnano和css-loader捆绑在一起，所以不需要自己加载它。不过你也可以使用postcss-loader显式的使用cssnano。有关于cssnano的详细文档，可以点击这里获取。

在cssnano的配置中，使用了preset: "advanced"，所以我们需要另外安装：

```
npm i cssnano-preset-advanced --save-dev
```

cssnano集成了一些其他的PostCSS插件，如果你想禁用cssnano中的某个插件的时候，可以像下面这样操作：

```
"cssnano": {
  autoprefixer: false,
  "postcss-zindex": false
}
```

上面的代码把`autoprefixer`和`postcss-zindex`禁掉了。前者是有重复调用，后者是一个讨厌的东东。只要启用了这个插件，`z-index`的值就会重置为1。这是一个天坑，千万记得将`postcss-zindex`设置为`false`。

### postcss-px-to-viewport
`postcss-px-to-viewport`插件主要用来把px单位转换为`vw`、`vh`、`vmin`或者`vmax`这样的视窗单位，也是vw适配方案的核心插件之一。

在配置中需要配置相关的几个关键参数：

```
"postcss-px-to-viewport": {
  viewportWidth: 750,      // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
  viewportHeight: 1334,    // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
  unitPrecision: 3,        // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
  viewportUnit: 'vw',      // 指定需要转换成的视窗单位，建议使用vw
  selectorBlackList: ['.ignore', '.hairlines'],  // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
  minPixelValue: 1,       // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
  mediaQuery: false       // 允许在媒体查询中转换`px`
}
```

目前出视觉设计稿，我们都是使用750px宽度的，那么100vw = 750px，即1vw = 7.5px。那么我们可以根据设计图上的px值直接转换成对应的vw值。在实际撸码过程，不需要进行任何的计算，直接在代码中写px，比如：

```
.test {
  border: .5px solid black;
  border-bottom-width: 4px;
  font-size: 14px;
  line-height: 20px;
  position: relative;
}
[w-188-246] {
  width: 188px;
}
```

编译出来的CSS：

```
.test {
  border: .5px solid #000;
  border-bottom-width: .533vw;
  font-size: 1.867vw;
  line-height: 2.667vw;
  position: relative;
}
[w-188-246] {
  width: 25.067vw;
}
```

在不想要把px转换为vw的时候，首先在对应的元素（html）中添加配置中指定的类名`.ignore`或`.hairlines`(`.hairlines`一般用于设置`border-width:0.5px`的元素中)：

```<div class="box ignore"></div>```

写CSS的时候：

```
.ignore {
  margin: 10px;
  background-color: red;
}
.box {
  width: 180px;
  height: 300px;
}
.hairlines {
  border-bottom: 0.5px solid red;
}
```

编译出来的CSS:

```
.box {
  width: 24vw;
  height: 40vw;
}
.ignore {
  margin: 10px; /*.box元素中带有.ignore类名，在这个类名写的`px`不会被转换*/
  background-color: red;
}
.hairlines {
  border-bottom: 0.5px solid red;
}
```

上面解决了px到vw的转换计算。那么在哪些地方可以使用vw来适配我们的页面。根据相关的测试：

* 容器适配，可以使用vw
* 文本的适配，可以使用vw
* 大于1px的边框、圆角、阴影都可以使用vw
* 内距和外距，可以使用vw

### postcss-aspect-ratio-mini
`postcss-aspect-ratio-mini`主要用来处理元素容器宽高比。在实际使用的时候，具有一个默认的结构

```
<div aspectratio>
    <div aspectratio-content></div>
</div>
```

在实际使用的时候，你可以把自定义属性aspectratio和aspectratio-content换成相应的类名，比如：

```
<div class="aspectratio">
    <div class="aspectratio-content"></div>
</div>
```

我个人比较喜欢用自定义属性，它和类名所起的作用是同等的。结构定义之后，需要在你的样式文件中添加一个统一的宽度比默认属性：

```
[aspectratio] {
  position: relative;
}
[aspectratio]::before {
  content: '';
  display: block;
  width: 1px;
  margin-left: -1px;
  height: 0;
}

[aspectratio-content] {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
}
```

如果我们想要做一个`188:246`（188是容器宽度，246是容器高度）这样的比例容器，只需要这样使用：

```
[w-188-246] {
  aspect-ratio: '188:246';
}
```

有一点需要特别注意：`aspect-ratio`属性不能和其他属性写在一起，否则编译出来的属性只会留下`aspect-ratio`的值，比如：

  `<div aspectratio w-188-246 class="color"></div>`

编译前的CSS如下：

```
[w-188-246] {
  width: 188px;
  background-color: red;
  aspect-ratio: '188:246';
}
```

编译之后：

```
[w-188-246]:before {
  padding-top: 130.85106382978725%;
}
```

主要是因为在插件中做了相应的处理，不在每次调用aspect-ratio时，生成前面指定的默认样式代码，这样代码没那么冗余。所以在使用的时候，需要把width和background-color分开来写：

```
[w-188-246] {
  width: 188px;
  background-color: red;
}
[w-188-246] {
  aspect-ratio: '188:246';
}
```

这个时候，编译出来的CSS就正常了：

```
[w-188-246] {
  width: 25.067vw;
  background-color: red;
}
[w-188-246]:before {
  padding-top: 130.85106382978725%;
}
```

有关于宽高比相关的详细介绍，如果大家感兴趣的话，可以阅读下面相关的文章：

* CSS实现长宽比的几种方案
* 容器长宽比
* Web中如何实现纵横比
* 实现精准的流体排版原理

> 目前采用PostCSS插件只是一个过渡阶段，在将来我们可以直接在CSS中使用aspect-ratio属性来实现长宽比。

### postcss-write-svg
postcss-write-svg插件主要用来处理移动端1px的解决方案。该插件主要使用的是border-image和background来做1px的相关处理。比如：

```
@svg 1px-border {
  height: 2px;
  @rect {
    fill: var(--color, black);
    width: 100%;
    height: 50%;
  }
}
.example {
  border: 1px solid transparent;
  border-image: svg(1px-border param(--color #00b1ff)) 2 2 stretch;
}
```

编译出来的CSS:

```
.example {
  border: 1px solid transparent;
  border-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' height='2px'%3E%3Crect fill='%2300b1ff' width='100%25' height='50%25'/%3E%3C/svg%3E") 2 2 stretch;
}
```

上面演示的是使用`border-image`方式，除此之外还可以使用`background-image`来实现。比如：

```
@svg square {
   @rect {
    fill: var(--color, black);
    width: 100%;
    height: 100%;
  }
}

#example {
background: white svg(square param(--color #00b1ff));
}
```

编译出来就是：

```
#example {
  background: white url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg'%3E%3Crect fill='%2300b1ff' width='100%25' height='100%25'/%3E%3C/svg%3E");
}
```

解决`1px`的方案除了这个插件之外，还有其他的方法。可以阅读前期整理的[《再谈Retina下1px的解决方案》](https://www.w3cplus.com/css/fix-1px-for-retina.html)一文。

> 特别声明：由于有一些低端机对border-image支持度不够友好，个人建议你使用background-image的这个方案。

## CSS Modules
Vue中的vue-loader已经集成了CSS Modules的功能，个人建议在项目中开始使用CSS Modules。特别是在Vue和React的项目中，CSS Modules具有很强的优势和灵活性。建议看看CSS In JS相关的资料。在Vue中，使用CSS Modules的相关文档可以阅读Vue官方提供的文档《CSS Modules》。

### postcss-viewport-units
`postcss-viewport-units`插件主要是给CSS的属性添加content的属性，配合viewport-units-buggyfill库给vw、vh、vmin和vmax做适配的操作。

这是实现vw布局必不可少的一个插件，因为少了这个插件，这将是一件痛苦的事情。后面你就清楚。

到此为止，有关于所需要的PostCSS已配置完。并且简单的介绍了各个插件的作用，至于详细的文档和使用，可以参阅对应插件的官方文档。

### vw兼容方案
在《再聊移动端页面的适配》一文中，详细介绍了，怎么使用vw来实现移动端的适配布局。这里不做详细的介绍。建议你花点时间阅读这篇文章。

先把未做兼容处理的示例二维码贴一个：

![image](https://upload-images.jianshu.io/upload_images/9159664-5a6257adc218af65..png?imageMogr2/auto-orient/strip|imageView2/2/w/136/format/webp)

你可以使用手淘App、优酷APP、各终端自带的浏览器、UC浏览器、QQ浏览器、Safari浏览器和Chrome浏览器扫描上面的二维码，您看到相应的效果：

![image](https://upload-images.jianshu.io/upload_images/9159664-66d70bb05e186b4c..png?imageMogr2/auto-orient/strip|imageView2/2/w/1078/format/webp)

但还有不支持的，比如下表中的No，表示的就是不支持

![](./image.png)

正因如此，很多同学都不敢尝这个螃蟹。害怕去处理兼容性的处理。不过不要紧，今天我把最终的解决方案告诉你。

最终的解决方案，就是使用viewport的polyfill：Viewport Units Buggyfill。使用viewport-units-buggyfill主要分以下几步走：

### 引入JavaScript文件
`viewport-units-buggyfill`主要有两个JavaScript文件：`viewport-units-buggyfill.js`和`viewport-units-buggyfill.hacks.js`。你只需要在你的HTML文件中引入这两个文件。比如在Vue项目中的index.html引入它们：

<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>

你也可以使用其他的在线CDN地址，也可将这两个文件合并压缩成一个.js文件。这主要看你自己的兴趣了。

第二步，在HTML文件中调用`viewport-units-buggyfill`，比如：

```
<script>
    window.onload = function () {
        window.viewportUnitsBuggyfill.init({
            hacks: window.viewportUnitsBuggyfillHacks
        });
    }
</script>
```

为了你Demo的时候能获取对应机型相关的参数，我在示例中添加了一段额外的代码，估计会让你有点烦：

```
<script>
    window.onload = function () {
        window.viewportUnitsBuggyfill.init({
        hacks: window.viewportUnitsBuggyfillHacks
        });

        var winDPI = window.devicePixelRatio;
        var uAgent = window.navigator.userAgent;
        var screenHeight = window.screen.height;
        var screenWidth = window.screen.width;
        var winWidth = window.innerWidth;
        var winHeight = window.innerHeight;

        alert(
            "Windows DPI:" + winDPI +
            ";\ruAgent:" + uAgent +
            ";\rScreen Width:" + screenWidth +
            ";\rScreen Height:" + screenHeight +
            ";\rWindow Width:" + winWidth +
            ";\rWindow Height:" + winHeight
        )
    }
</script>
```

具体的使用。在你的CSS中，只要使用到了viewport的单位（vw、vh、vmin或vmax ）地方，需要在样式中添加`content`：

```
.my-viewport-units-using-thingie {
  width: 50vmin;
  height: 50vmax;
  top: calc(50vh - 100px);
  left: calc(50vw - 100px);

  /* hack to engage viewport-units-buggyfill */
  content: 'viewport-units-buggyfill; width: 50vmin; height: 50vmax; top: calc(50vh - 100px); left: calc(50vw - 100px);';
}
```

这可能会令你感到恶心，而且我们不可能每次写vw都去人肉的计算。特别是在我们的这个场景中，咱们使用了postcss-px-to-viewport这个插件来转换vw，更无法让我们人肉的去添加content内容。

这个时候就需要前面提到的`postcss-viewport-units`插件。这个插件将让你无需关注content的内容，插件会自动帮你处理。比如插件处理后的代码：

![image](https://upload-images.jianshu.io/upload_images/9159664-c8ce5d8618b11c24..png?imageMogr2/auto-orient/strip|imageView2/2/w/784/format/webp)

`Viewport Units Buggyfill`还提供了其他的功能。详细的这里不阐述了。但是content也会引起一定的副作用。比如img和伪元素`::before(:before)`或`::after（:after）`。在img中content会引起部分浏览器下，图片不会显示。这个时候需要全局添加：

```
img {
  content: normal !important;
}
```

而对于`::after`之类的，就算是里面使用了`vw`单位，`Viewport Units Buggyfill`对其并不会起作用。比如：

// 编译前
```
.after {
  content: 'after content';
  display: block;
  width: 100px;
  height: 20px;
  background: green;
}
```

// 编译后
```
.after[data-v-469af010] {
  content: "after content";
  display: block;
  width: 13.333vw;
  height: 2.667vw;
  background: green;
}
```

这个时候我们需要通过添加额外的标签来替代伪元素（这个情景我没有测试到，后面自己亲测一下）。

到了这个时候，你就不需要再担心兼容问题了。比如下面这个示例：

![image](https://upload-images.jianshu.io/upload_images/9159664-fb229770cb3f8599..png?imageMogr2/auto-orient/strip|imageView2/2/w/136/format/webp)

请用你的手机，不管什么APP扫一扫，你就可以看到效果。（小心弹框哟），如果你发现了还是有问题，请把弹出来的信息截图发给我。

如查你想看看别的机型效果，可以点击这里、这里、这里、还有这里。整个示例的源码，可以点击这里下载。

如果你下载了示你源码，先要确认你的系统环境能跑Vue的项目，然后下载下来之后，解压缩，接着运行`npm i`，再运行`npm run dev`，你就可以看到效果了。

> [如何在Vue项目中使用vw实现移动端适配](https://www.jianshu.com/p/1f1b23f8348f)
> [如何在Vue项目中使用vw实现移动端适配](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)
