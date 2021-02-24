## Vue.js 学习笔记

[TOC]

### H5和ES6基础知识拾遗

- 从var到let&const：

  - var可以多次声明；let和const不可多次声明
  - var只有全局和函数作用域（在ES5中使用闭包，定义一个函数作用域以保护var不受外界影响，也不被函数内部影响）；let和const有块级作用域 （<u>**{ }**</u>，以及if、for等），在block外无效，这就没有变量提升和污染全局的危险（实例：for的底层原理和ES5中的闭包）
  - 能使用const，尽量使用const；const在定义的时候就要初始化；const的对象不可以修改（指定了一个内内存地址），但是属性值可以修改（例如key不可以改，value可以）

- 对象字面量的增强写法

  ```javascript
  //ES5
  const var1 = "12";
  const var2 = "34";
  const var3 = '56';
  const total = {
    var1: var1,
    var2: var2,
    var3: var3,
    func: function () {}
  }
  //ES6
  const var1 = "12";
  const var2 = "34";
  const var3 = '56';
  const total = {
    var1,
    var2,
    var3,
    func () {}     //可以看到ES6的字面量定义，不管是属性还是方法都简单了很多
  }
  ```

- ES6箭头函数

  ```javascript
  //ES6的一种函数定义方法，它常用在一个函数作为另一个函数的参数的时候
  const func = (/*参数放在这里*/) => {
    //函数内容
  }
  //如果函数只有一个参数，可以不要小括号；如果函数只有一行代码，可以省略花括号，也不需要return
  const func2 = num => num*3;
  const result = func2(3);
  ```

  

### Vue基础知识

此略，见官方文档、菜鸟教程和印象笔记，这里仅做一些补充

- 概念
  - MVVM：模板（Model）、视图（View）和视图模板（ViewModel）。在Vue中，模板是js代码和后端传入的数据；视图是网页的DOM，而视图模板则指vue提供的框架和方法，它将前端dom和后端的方法联系起来，实现视图响应和数据同步。
  - 命令式编程与声明式编程：传统jQuery采用命令式编程，而现在流行的框架则采用声明式编程。
  - 面向对象编程的一些概念：类、实例、方法（vs函数，声明函数，调用方法，方法与实例绑定）
  - Mustache语法：{{就是这个}}。它支持一般的插值和简单的表达式

- Vue Options
  - el: String（id）| HTML
  - data: 对象（String）| function
  - methods
  - computed...

- Vue生命周期

  Vue实例不是静态的、长期霸占内存的。在Vue源码中规定了每一个Vue实例从创建到销毁的一系列动态过程，实例中的options都在这一个生命周期中以一定的顺序被使用。官网上可以查询到Vue的标准生命周期。

  我们可以在了解Vue生命周期的基础上自定义一些Hooks，相当于自定义Vue实例的一些流程。

  ```javascript
  new Vue ({
    <options> //常规option，比如el, data
    lifeCycle: function () {}
  	//声明一个生命周期阶段，例如beforeCreate，mounted，然后说明方法，就形成一个hook
  })
  ```

  例如，在某一阶段中添加一个`console.log`的方法，可以帮助我们确定实例正常运行的时序；添加一些网络服务，可以动态调用后台数据等。

- 虚拟DOM与元素复用

  Vue会先根据代码渲染生成一个虚拟DOM，再将虚拟DOM渲染到浏览器中，在这个过程中为了提高性能，存在一些元素的复用。例如，设置一对互斥的、可切换的input标签（通过v-if等方法），切换时input及其内容会保留。如果不希望元素复用，可以给复用元件设置不同的key属性`<div key='1'></div>`

- el: el在底层是一个mount（挂载），其指向的内容会被render覆盖。

### 常见Vue基本指令

  - v-bind:<feature>=""：绑定属性

  - v-on: <event> / @<event>:监听事件`<button @click="num++">ADD</button>`

  - V-for=" (<item>, <index>) in <items>": 循环指令，一般item是视图中的变量，items是由实例规定的数组

  - v-html="<url>"：如果要用Vue实例实现html标签的解析，需要使用v-html（直接加载字符串不可以解析），这个不需要在DOM中额外添加变量。`<div v-html="url"></div>`

  - v-text=''<meaasge>"":类似于v-html, 是可以无变量显示文本的命令（覆盖原html标签）**（没啥用）**

  - v-once: 添加这个指令以后，该组件不会随着实例的变化而变化（只加载一次）。这个指令没有其他参数

  - v-cloak: 为了避免插值闪烁（Vue代码延迟解析导致原DOM暴露），添加这个属性以后，联合css：

    ```css
    [v-cloak]: {
    	display:none;
    }
    ```

    可以隐藏未被解析的DOM。这个指令没有其他参数

  - v-pre：添加这个指令后，可以将标签内的内容转义，不会被Vue实例解析。这个指令没有其他参数

#### v-bind的使用

属性动态绑定，在v-bind标签中的属性会是可由Vue实例编辑的对象

语法糖：`:<feature>=""`

```HTML
<img v-bind:src='url' id='test'></img>
<script>
  new Vue ({
    el: '#test',
    data: {
      url:""
    }
  })
</script>

<a :href='www.baidu.com' id="hrefTest">超链接</a>
<script>
  new Vue ({
    el: '#hrefTest',
    data: {
      url: String //这里的String没有引号，它是对象而不是字符，它规定了标签解析的方法
    }
  })
</script>

<div :class="{class1: value1, class2: value2}" @click='change()' id='test3'>TEST</div>
<!--class是我们预置的样式类，value则是布尔值。value的TF决定了标签采用什么类（当然可以有多个TRUE）。value可以通过方法或js直接改变实例的值来调整。-->
<!--如果觉得class里的内容太长，可以在class里写一个函数style()，由methods return这个对象-->
<!--class里还可以放一个数组[class1, class2]，一次储存几个class，当然这个一般的class就没啥区别了……-->
<!--v-bind：style='{key:value, key:value}'，这里的对象key是css属性，而value是具体的属性变量/'属性值'-->

<script>
  new Vue ({
    el:'#test3',
    data: {
    	value1: true;
    	value2: false
  	},
    methods: {
    	change: function () {
    		this.value1=!this.value1;
    		this.value2=!this.value2;
  			}  //TF交换；方法使用实例内变量（包含DOM内声明的变量）都需要用this.
  	}
  })
</script>
```

#### v-on的使用

事件监听标签

语法糖`@<event>='function(<parameter>)'`

- 常见事件：
  - @click
  - @keyup: 键盘按键
- v-on修饰符：@event.修饰符
  - .stop：可以阻止与该组件图层叠加的其他组件的事件，可以防止事件冒泡
  - .prevent：可以阻止该动作的默认事件，只执行实例规定的方法
  - .{keycode}：只相应特定键盘按键的事件`@keyup.{enter}`
  - .once：只触发一次事件

> v-on可以用来监听浏览器传递的event对象，参数名固定为$event

#### 逻辑结构：v-if, v-else-if和v-for

分支逻辑不是很常用，可以使用methods或者computed方法实现逻辑

- v-if和v-else配合，直接加入标签时可以根据v-if的布尔值决定show or not；v-else-if就相当于分支选择，还是根据布尔值来判断显示哪个分支（可以在标签中显示判断式）

  ```html
  <div id="app">
    <div v-if="judge">
      True
    </div>
    <div v-else>
      False
    </div>
  </div>
  <script>
  	new Vue ({
      el: "#app",
      data: {
        judge: true,   //若为false则显示v-else的内容
      }
    })
  </script>
  ```

- v-if 和v-show的区别：使用v-show，若逻辑判断为false，标签会隐藏但依然存在于DOM中，而v-if若为false，标签直接消失；频繁切换时，v-show的性能更高，而v-if使DOM更为简洁

- v-for

  ```html
  <div id = "app" v-for="{item, index, key, value} in items">
  <!--可以遍历的包括（数组的）对象、（数组的）索引、（对象的）键、（对象的）键的值-->
    {{}}
  </div>
  <script>
    new Vue ({
      el: "#app",
      data: {
        items: {
          
        }
      }
    })
  </script>
  ```

  如果要提高遍历性能，可以给v-for标签添加key属性`key = "item"`, 使key和遍历对象唯一对应。在Vue底层代码中，key是虚拟DOM的唯一标识。如果在一次遍历以后修改数组再遍历一次，使用key标识的对象会直接复用（否则会重新渲染）。

#### v-model的使用

#### computed的使用

**计算属性**，本质上是定义一个属性，通过get和set方法，效果是直接将模板中的变量定义为表达式（return）。

```html
<!--conputed的完整内容-->
<script>
  new Vue ({
    computed: {
      attribute: {
        get: function () {   //一般我们只定义get方法，可以简写（如下例）；只定义get，attribute就是只读属性；其中也可以加一些动态命令（比如console.log()）
          
        },
        set: function (input) {   //一般我们不需要定义set方法，直接删除即可；如果写了内容，就可以对输入的值产生响应（例如在console里给attribute赋值）
          
        }
      }
      }
    }
  })
</script>
```

``` html
<div id="test">
  {{msg}}
</div>
<script>
  new Vue ({
    el: "#test",
    data: {
      items: [
        {key: 1, value: 123},
        {key: 2, value: 234}
      ]
    },
    computed: {
      msg: function () {								//简写的语法糖，只包含get方法
        let result = 0;
        for (let i in this.items): {    //ES6语法
          result += this.items[i].value
        };
        return result;
      }
    }
  })
</script>
```

与直接return一个methods的结果相比，computed的特点是**使用缓存**。如果多次调用这个方法/属性，methods会调用计算多次，而computed在输入不变的前提下只计算一次（输入变了也会重新计算）。总的来说，computed的性能更好。

### Vue的组件化开发

#### 组件的创建和注册

#### 全局组件、局部组件和父子组件

#### 组件数据和父子数据传递

---------

### Webpack基础
Webpack是js应用的模块化打包工具，它也vue cli使用的打包工具
Webpack可以将复杂的js项目（甚至是包含多种标准的js项目）转化为浏览器便于识别的代码；它的主要作用是模块整理（梳理整合模块关系）和浏览器保证

#### 项目开发和打包（基础）
webpack最大的优势是对模块化开发标准的支持，包括CommonJS，AMD，CMD和ES6等。项目开发的时候可以使用任意的模块化开发标准，也可以混杂使用多种标准

**既可以使用webpack自带的框架在命令行中创建项目，也可以对自己创建的项目使用webpack命令进行打包**

```powershell
webpack src/main.js -o dist/index.js 
#确保webpack可以全局使用；前一个是入口文件，后一个是出口文件；适用于webpack4
```

#### 安装

基于node，使用npm就可以安装`npm install webpack -g`

（mac可能需要sudo，webpack4需要webpack-cli）

**全局安装与本地安装**：在家目录下安装webpack常常使用全局安装，但是具体项目中未必可以使用全局安装的版本（例如项目都是用比较早期的 webpack版本做的），这时候如果要做开发就要在项目根目录里单独安装对应的webpack版本（进入项目根目录使用npm install，指定版本，不要-g），这就是本地安装。

#### Webpack配置文件

每次打包都要写入口和出口非常麻烦，我们可以设定配置文件来实现更简单的打包

在项目（根目录）中穿件webpack.config.js文件，可以在里面设置：

```javascript
const path = require('path')

module.exports = {
  entry: '',        //入口路径。入口对模块的依赖是webpack打包的基础，它应该声明所有需要的模块路径（无脑import），当然也可以执行一点js功能
  output: {
    path: path.resolve(__dirname, ’dist‘), //出口路径，需要是一个绝对路径；这里使用了node的path package动态生成一个拼接路径
    filename: 'bundle.js',   //打包后的文件名称（一个js文件）
	}
}
```

设置完毕以后，**在根目录中执行`webpack`就可以根据设定打包**（或者在设定package.json以后使用npm run执行打包）

package.json文件也在根目录下，它包含了该项目的一些基本信息，以及依赖包的信息。在分享项目的时候，被分享者就可以根据package.json中的内容安装npm中的其他package。如果是自建的项目没有package.json文件，可以在根目录下使用`npm init`命令初始化项目设置并且生成这个文件。

对package.json文件的部分说明：

```javascript
{
  "scripts": {    //这里可以定义一些脚本，在命令行中使用 ’npm run 脚本名‘ 实现
    "test": "echo 'ERROR......' "，
    "bulid": "webpack"，         //在命令行中执行npm run bulid 相当于执行webpack，而且这样子会优先使用本地webpack进行打包（而不是全局webpack）
  }
}
```

#### loader的使用

webpack原生仅支持js的打包，而使用loader就可以扩展对css和其他数据的打包。在webpack官网，根据对文件的需要可以查看对应loader的使用教程。基本方法是：

1. 安装：loader本质上还是node包。在项目中使用npm install 本地安装对应的loader。注意版本号。

2. 配置：在webpack.config.js中添加loader设置：

   ```javascript
   module.exports = {
     module: {
       //这里复制官网代码即可orz
       test: '',  //使用正则表达式匹配文件类型
       use: ['',''], //对于这种文件类型使用哪些loader，它会从右向左使用loader
     }
   }
   ```

3. 声明依赖关系：例如，如果要打包css文件，需要在入口js中添加对css文件的依赖`requires('[path]')`
4. webpack打包即可

#### 在Webpack中配置Vue

##### 引入Vue package
在Webpack（和Vue Cli）中，一般我们会以npm package的方式模块化引入Vue（而不是原生开发的CDN或本地js）。
1. 在项目根目录中安装Vue `npm install vue --save`
2. 在入口js文件中声明依赖 `import Vue from 'vue'`

一般来说，这样就可以在webpack的项目中使用Vue。做到这一步，就相当于原生html中引入了Vue的CDN

##### Webpack中的组件化开发（对于单页面应用SPA）

在SPA中，以一般规范而言，我们在index.html中只会包含一个div标签，而div里的所有内容都会放在entry.js的Vue实例中，如下：

```javascript
import app from "app.vue"  //可以从其他js导入一个组件~(或者是从vue导入)
const app = new Vue ({
  el: '',  //还是需要通过el连接到index
  template: {      //在Vue实例中添加了template，这个template会直接加载到index.html文件中el对应的标签中（Vue实例就是组件，组件就是Vue实例） 
    `<cpn></cpn>`
  },
  components: {
    app            //某个另外创建的组件，这个组件完全可以放在另一个文件中，并且通过import导入；当然也可以在这个文件中自己写
  }
})
```

##### .vue文件进行组件化开发

有了vue本身的组件思想和webpack提供的模块集成，我们就可以比较整洁地进行模块化开发。

创建一个app.vue文件。这个文件在规范上就是index.html中的大页面（相当于把body的所有DOM总结到一个div中，再把其中的代码全都转移到app.vue中）

```vue
<template>
	<div>
    <!--这里填写组件的html代码，可以包含一般Vue组件的功能-->
  </div>
</template>

<script>
  //如果从其他文件获取子组件，就从这里导入
  export default {
    name: "app", //定义一个name，供其他文件import
    methods: {},
    data() {},        //字面量的增强写法，相当于data: function() {}，定义了一个名为data的方法
    componenets: {},  //规定这个组件的数据、方法和子组件等
  }
</script>

<style>
</style>
```

在入口中可以`import app from ’app.vue‘`

然后需要配置vue-loader和vue-template-compiler

最后使用webpack打包

##### Webpack的plugin

Webpack支持很多plugin，可以扩展webpack的功能。这里我们重点介绍一个插件，**HtmlWebpackPlugin**. 使用这个插件以后，我们才可以方便地把打包后的文件部署到服务器上。

> webpack2一下打包后的文件都只存在于dist文件夹，但是只包含js/css等，我们开发时使用的index.html文件不会被打包（离谱）。使用这个插件以后，就可以自定义创建打包后的html文件，这同时也是开发多页面应用的基础之一。

把HtmlWebpackPlugin安装到项目后，在webpack.config.js中配置：

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = {
  entry: '',          //还是需要设置js等文件的打包
  output: {
    path:
    filename: ''
  }
  new HtmlWebpackPlugin ({
  template: '',    //这里可以导向一个html文件的路径，打包后新生成的html文件会以这个html为模板（例如都包含<div id="app"></div>，都具有head等）。我们直接将src中的html文件作为模板，就可以连接开发与打包后的html文件。
  filename: ''，   //定义打包后的html文件名
  //有多个页面就创建多个实例
})
}
```

##### Webpack-dev-server的本地服务器

Webpack-dev-server也是一个plugin。使用这个命令（`npm run start`）以后，我们就可以创建一个node环境、express内核的本地服务器（数据储存于内存）并进行热更新。再次使用就会存储到硬盘。在Vue cli中是内置这个插件的。

### Vue Cli基础

脚手架适于大型项目的开发，它是一套基于webpack的模板，可以帮助我们生成适于Vue开发的一个webpack环境。它的主要优势就是帮开发者简化Vue的webpack开发的配置过程。

#### 安装和项目创建

见印象笔记

#### 配置

Vue Cli 3 及以后的版本主张”零配置“（可以降低入门难度），在新建项目后可以看到项目目录里没有了Vue Cli 2 的那么多配置文件，大多数配置都缺省了。修改配置的方法：

1. 使用Vue ui创建项目（而不是vue create），这样会生成一个本地服务，以GUI网页的模式自定义项目新建。Vue ui也可以管理已经创建的项目（导入功能）
2. node_modules/@vue/cli-service/lib内有许多配置文件夹，如果看得懂就可以修改orz
3. 自定义配置，在根目录下创建vue.config.js文件夹，可以在其中输入自定义配置

#### Vue UI的使用

Vue ui有丰富的功能，包括：

- 新建项目
- 查看、编辑和新安装插件和依赖（这就不需要npm的命令行操作和手动修改配置文件了）
- 执行serve、build、lint等命令（比命令行操作更方便好看）
- 设置一些webpack配置（webpack.config.js），包括公共路径等

### 路由Router基础

路由（Routing）是网络工程术语，它指数据的分配转送过程。

互联网--公网IP--局域网IP（运营商交给调制解调器）--内网IP（路由器交给设备，路由器将内网IP与设备的MAC地址联系，形成路由表）

#### 前端渲染与后端渲染、前端路由与后端路由

在web应用中，路由指资源请求与页面展示的关系，一般可以理解为url和页面的映射关系。

- 在web应用的最早期，由于前端计算力很差，采用完全后端渲染的模式。服务器接受url请求后，通过jsp/php/asp等获取数据库数据并直接生成固定的html+css网页发送给用户。我们检查网页的时候可以看到网页里写满了在后端动态生成的html代码和数据。这个时候实际上没有前端的概念，所有内容都是后端工程师的工作。在后端渲染模式中，服务器接收url，根据url生成页面并将成熟的页面发送给用户，这就是后端渲染和后端路由。
- 纯后端渲染的模式形成的代码非常混乱，一个html文档中包含html、css和java代码，难以维护。后来，以ajax等为基础，出现了前后端分离的模式。前端发送url以后，从静态资源服务器获得部分html、css和js代码，浏览器在读取js代码的过程中，发现了ajax等请求，根据ajax提供的api接口到api服务器上获取实时数据，传回前端，利用前端的js代码控制网页。在这种模式中，网页的实际内容是完全由前端的js代码形成的，**前端负责html和js设计，后端负责整理数据并提供ajax的api接口**。这就是前端渲染。

- 现在非常流行的SPA模式，以一个html网页包含所有内容。一个url会从服务器请求到所有的资源（除了一些懒加载的异步资源），这实际包含了多个页面的内容。而在用户端，用户每次只能看到一个网页，不同网页视图之间的切换是通过前端的映射关系实现的，这就是前端路由。

#### hash和history mode

前端路由的一个特点就是url变化不会导致页面刷新，这可以通过url的hash与H5的history mode两种方式实现。

- 在浏览器console中输入`location.hash = 'foo'`，可以发现url改变了，但是浏览器没有刷新（没有请求新的资源），vue-router可以通过这种方式监听hash来控制页面**（默认模式）**

- 浏览器console键入`history.pushState({},'','foo')`也可以发现url改变了但是页面没有刷新(栈结构)；键入`history.repalceState({},'','bar')`也可以改变url而不刷新，它是替换的结构

  ```cmd
  #栈结构的方法
  history.pushState({},'',"foo")
  history.back()
  history.forward()
  history.go(#数字#)
  
  #替换结构的方法
  history.replaceState({},'','foo')
  ```

#### Vue-Router配置和使用

各个前端框架都有自己的router plugin，vue就是vue-router。它可以用webpack导入，也可以在Vue Cli创建项目的时候引入。

在项目里有一个router文件夹，index.js（注意，不是入口，入口现在叫main.js）。

```javascript
import { createRouter, createWebHashHistory } from 'vue-router'  //导入插件
import Home from '../views/Home.vue'                             //导入总组件

const routes = [                                                 //配置路由属性
  {
    path: '/',                                                   //一个映射的基本属性包括path和component;这里不加路径就是默认首页
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../views/About.vue')
  }，																															//箭头函数
  {
  	path: '/home',
  	redirect: '/',																								//路由的重定向
  }
]

const router = createRouter({
  history: createWebHashHistory(),                              //这里设置了history模式，它与默认的hash模式各有优劣
  routes                                                        //引用刚才配置的属性创建路由            
})

export default router                                           //把创建出的router导出，加入Vue实例中（在entry）

```

前端路由的实质就是监听url或者history的变化。在router文件夹中我们设定好了路由的映射关系并且导出了，而在显示层，我们需要设置触发器来出发url或者history的变化。在原生开发中，`<a>`标签可以改变url，而在vue-router中，我们使用router-link标签

#### Router-link的使用方法

```vue
//在显示层添加标签（可以在根组件App.vue里添加）
<template>
	<div id = "app">
    <router-link to = "/">Home</router-link>                 <!--router-link是Vue内部注册的组件，它最后其实被渲染成了a标签，修改了url-->
    <router-link to = '/about'>About</router-link>
    <router-view/>                													<!--router-view标签是router渲染出组件的占位符，它告诉页面组件应该出现在哪-->        
  </div>
</template>

<script>
  //router-link的一些属性：
  //1. 可以作为其他标签（而不渲染成a href形式）
  //2. 当前被渲染的router-link有active属性，可以利用
  //3. replace属性，可以把这个陆游与从push（堆栈，可返回)变为替代（不可返回）
  
  //通过js实现路由跳转（相当于通过函数设定按钮事件），也就是js修改url
  function () {    //在组件化开发中，它经常作为组件的method
    this.$router.push('<path>')  //$router是vue-router自带的实例变量，push代表加一段url；也可以用replace等
  }
</script>
```

