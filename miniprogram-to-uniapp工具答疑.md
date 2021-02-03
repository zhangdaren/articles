# miniprogram to uniapp 工具答疑

## 前言
在群里给大伙答疑时，发现遇到的问题仍然不少，有工具无法处理的问题，也有代码本身的问题，因此在这里列出遇到的问题及解决方案，供转换时参考。   

在这之前还是要声明，小程序语法与vue语法无法做到一一对应，工具现在无法做到100%转换，或多或少需要做一些修改   

但有例外，RMB可以~   

建议通过本工具转换为uniapp项目后，将转换后的项目导入到hbuilder X里，**先编译为微信小程序，看看是否可以正常编译和运行，保证 “微信小程序-->uniapp-->微信小程序” 是ok的，然后再尝试编译到其他平台。**

有报错是正常的，请参考下面文档进行修改，或加入qq群： 780359397 或 361784059 进行讨论和建议，感谢支持！

<a target="_blank" href="http://shang.qq.com/wpa/qunwpa?idkey=6cccd111e447ed70ee0c17672a452bf71e7e62cfa6b427bbd746df2d32297b64"><img border="0" src="http://pub.idqqimg.com/wpa/images/group.png" alt="小程序转uni-app讨论群" title="小程序转uni-app讨论群"></a>

有时编译报错，但又不知道哪行报错，可以在工具--插件安装，将eslint两个插件安装上，然后在页面里右键---验证本文档语法，即知道哪行代码有问题了。

注意：**如果是vant项目，转换后的uniapp项目仅支持app和h5平台**，相对来说，调试起来稍为繁琐和麻烦。


## 安装时，一直停留在fetchMetadata: sill……
安装npm install时，长时间停留在fetchMetadata: sill mapToRegistry uri   

你可以首先检查一下你的源是不是淘宝的镜像源   

```
npm config get registry
```

如果不是的，更换成淘宝的源，执行下面的命令：  

```
npm config set registry https://registry.npm.taobao.org
```

配置后再通过下面方式来验证是否成功   

```
npm config get registry
```

然后再执行：

```
npm install miniprogram-to-uniapp -g
```


## 无法加载文件 XXXXXXXXX.ps1，因为在此系统上禁止运行脚本。
以管理员身份运行powershell   

执行   
```
set-executionpolicy remotesigned
```

输入y即可   


## 运行wtu -V报错   
```
$ wtu -v   
/usr/local/lib/node_modules/miniprogram-to-uniapp/src/index.js:297   
async function filesHandle(fileData, miniprogramRoot) {   
^^^^^^^^   
SyntaxError: Unexpected token function   
......   
```
#### 错误原因：
当前nodejs版本不支持es6语法   
#### 解决方案：
升级nodejs版本，建议v9以上   



## mac安装报错Error: EACCES: permission denied, access '/usr/local/lib/node_modules
#### 错误原因：
执行命令行命令时没有获得管理员权限   
#### 解决方案：
在命令行前面添加sudo获取管理员权限，输入管理员密码就行   
sudo npm install miniprogram-to-uniapp -g


## 运行到H5或其他平台，控制台报错：[system] API `login` is not yet implemented   
#### 错误原因：   
此api在当前平台不支持，具体请查看uniapp的api文档   
#### 解决方案：   
绕过，使用条件编译不同平台，使用不同的处理方式。   
如，登录，h5里的登录涉及到很多种情况，需自行处理，如使用帐号密码、手机号验证码、微信登录等等方式。   


## 报错：[Vue warn]: Invalid prop: type check failed for prop "couponList". Expected Array, got String with value "".
#### 错误原因
变量couponList定义的时候是Array，但给它赋的值是String，这种写法在uniapp没法通过编译。

#### 解决方案
将该变量的需要给它的值的类型修改一下，或者将定义的类型调整为String。

#### 重现示例
```
//coupon-window.vue
<template>
    <view></view>
</template>

<script>
export default {
  data() {
    return {};
  },
  props: {
    couponList: {
      type: Array,
      default: () => []
    }
  }
}
</script>
```

```
//index.vue
<template>
    <view>
        <coupon-window :couponList="couponList"></coupon-window>
    </view>
</template>

<script>
import couponWindow from "../components/coupon-window/coupon-window";
export default {
  data() {
    return {
      couponList: "",  //修改点：将这里调整为[]
    };
  },components:{
      couponWindow
  }
}
</script>
```
#### 修复示例
见示例中注释。



## 报错：[Vue warn]: Avoid adding reactive properties to a Vue instance or its root $data at runtime - declare it upfront in the data option.
#### 错误原因
试图对data里一个不存在的变量赋值时，这里会有两种情况：   

第一种：data里不存有abc变量时，使用```setData({abc:""})```就会报错，微信小程序是可以接受变量未在data里定义时进行赋值；   
第二种：data里不存在abc变量，但props里存在abc变量。微信小程序里，data和props是一个对象，但uniapp/vue里不是，而且在子组件里也无法对props里的变量进行赋值，所以也会报错。
>https://www.jianshu.com/p/422a05e2f0f4   
>其实在小程序里，properties和data指向的是同一个js对象，换一种说法，我们可以理解为：小程序会把properties对象和data对象合并成一个对象。   
>所以我们得出一个结论：我们不要把data和properties里的变量设置成同一个名字，如果他们名字相同，properties里的会覆盖data里的。
   


#### 解决方案
针对上面两种情况，解决方案如下：

第一种：工具尽可能的收集所有setData里的变量，检测是否定义过，但仍然会存在有漏网之鱼。
如在页面里将this通过某方法传递到App.vue里，然后在App.vue里的函数里调用这个传递过的this进行setData()时，工具无法针对这种情况进行修复，需手动在该页面添加变量。

第二种：在data里新建一个变量newVal，使用watch监听原来的变量变化时，就将新值赋值给这个变量newVal，并且使用这个newVal接管原来那个变量的所有操作。



#### 重现示例
  
```
<template>
    <view>
        <view :html="article"></view>
    </view>
</template>

<script>
export default {
  data() {
    return {}
  },
  props: {
    article:String,
  },
  methods: {
   setClass: function () {
      var content = '<p>hello world</p>';
      this.setData({
        'article': content
      });
    }
  }
}
</script>
```
#### 修复示例

```
<template>
    <view>
        <!-- 接管原来的变量 -->
        <view :html="articleBak"></view>
    </view>
</template>

<script>
export default {
  data() {
    return {
        articleBak:""   //新建一个变量
    }
  },
  props: ["article"],
  watch: {
    //监听props里的变量
	article:{
		handler(newVal, oldval) {
		   this.articleBak = newVal;
		},
		immediate: true,  //刷新加载 立马触发一次handler
		deep: true  // 可以深度检测到 person 对象的属性值的变化
	}
  },
  methods: {
   setArticle: function () {
      var content = '<p>hello world</p>';
      this.setData({
        'articleBak': content   //赋值给新变量
      });
    }
  }
}
</script>
```
## 报错：index.umd.min.js?1c31:1 [system] TypeError: Cannot use 'in' operator to search for 'class' in undefined
解决方案与上面一致，因为是上面的报错引起的。
    
```
    index.umd.min.js?1c31:1 [system] TypeError: Cannot use 'in' operator to search for 'class' in undefined
    at VueComponent.set [as $set] (vue.runtime.esm.js?e143:1087)
    at eval (main.js?4675:20)
    at Array.forEach (<anonymous>)
    at eval (main.js?4675:18)
    at Array.forEach (<anonymous>)
    at VueComponent.setData (main.js?4675:14)
    at VueComponent.setClass (index.vue?fff9:96)
    at VueComponent.mounted (index.vue?fff9:43)
    at invokeWithErrorHandling (vue.runtime.esm.js?e143:1865)
    at callHook (vue.runtime.esm.js?e143:4276)
```


## 语法错误: This experimental syntax requires enabling the parser plugin: 'dynamicImport'  

#### 错误原因
可能是函数名使用了系统保留关键字，如```<input @input="import"></input>```   

##### 同类错误：SyntaxError: Unexpected keyword 'class' (13:12)  
##### 同类错误：unexpected token default

#### 解决方案
工具尽可能会对该页面里使用到函数的所有地方进行重名。
此操作是有风险的，如果该页面在别的页面进行引用，并且调用了该函数，那这种情况工具将无法进行替换，需在项目报错时手工判断并替换。

#### 重现示例
  
```
<template>
    <view>
       <!-- 重名 -->
       <input @input="import"></input>  
    </view>
</template>

<script>
export default {
  data() {
    return {}
  },
  onload(){
    this.import();  
  },
  methods: {
   import: function () {
     console.log("hello");
    }
  }
}
</script>

```
#### 修复示例  
```
<template>
    <view>
       <input @input="importFun"></input>  
    </view>
</template>
<script>
export default {
  data() {
    return {}
  },
  onload(){
    this.importFun();  //重名
  },
  methods: {
   //重名
   importFun: function () {
     console.log("hello");
    }
  }
}
</script>

```


## 通过selectComponent(selector)选择的页面，在使用setData时，需注意    

解决：仍需手动调整处理。   

``` javascript
var pages = getCurrentPages();
var ctx = pages[pages.length - 1];
var pageCtx = ctx.selectComponent(selector);
pageCtx.setData({show:true}); 

```

上述代码报错：   
[Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. 
Instead, use a data or computed property based on the prop's value. Prop being mutated: "show"   

   
正确调用方式：   
``` javascript
var pages = getCurrentPages();
var ctx = pages[pages.length - 1];
var pageCtx = ctx.selectComponent(selector);
pageCtx.$vm.setData({show:true}); 

```
   
使用this来选择又不一样了，上面是使用当前页面的方式   

``` javascript
var o = e().selectComponent("#top-tips");

const nav = target.selectComponent('.nav-instance')

this.selectComponent("#app-share").showShareModal()

this.uploader = this.selectComponent("#uploader"); 
```
需换成```this.$scope.selectComponent();```



倒是可以一刀切：   
```
ctx.selectComponent(selector);     
替换为
ctx.selectComponent(selector).$vm;   
```
暂未并入工具，现在手动修改。   
（原因是ctx.selectComponent(selector)为null时，会报错Cannot read property '$vm' of null）   

---

2020-08-29 最新方案：
建议使用vue调用组件的方式，即：
```
<diy-compoent ref="abc"></diy-compoent>
 
 //通过refs获取到abc这个组件，并调用它的fun方法
this.$refs.abc.fun();
```


## 返回顶部不生效   
可参考：[scroll-view](https://uniapp.dcloud.io/component/scroll-view)里的写法   

同样也可以简单粗暴的修改代码为：```this.scrollTop = Math.random();```
原理见：[scroll-view组件返回顶部不生效！（附第三方解决方案）](https://ask.dcloud.net.cn/article/36612)


## 编译报错：不支持动态插槽
#### 错误原因
uni-app非v3模式不支持动态插槽

#### 解决方案
#%#&（*￥#*￥

#### 重现示例
  
```
<solt :name="tabItem"></solt>
```
#### 修复示例 
#%#&（*￥#*￥


## 语法错误：v-if=""   

#### 错误原因
v-if没有值   
附原小程序代码：
```
<view wx:if="{{}}"></view>
```
#### 解决方案
#&*（%￥#&￥（%#

#### 重现示例
  
```
<view v-if=""></view>
```
#### 修复示例 
```
<view></view>
```


### 语法错误：引号或括号不匹配   
如
```
<view style="line-height: 48rpx\""></view>
<view style="width:{{percent}}% }};"></view>
```
原始代码就有问题，需自己手动调整。   

### UnhandledPromiseRejectionWarning: TypeError: Cannot read property 'buildError' of undefined
原因：   
可能是因为代码里，let和var对同名变量先后进行了声明导致。   
如：   
```
let a = 1;
var a = 4;
```

### bindtap里为长代码   
如
```
 <a bindtap="{{canBuy==''&&buyType=='cart'?'getCart':''}}{{canBuy==''&&buyType=='buy'?'buyNow':''}}{{canBuy==''&&buyType=='select'?'select':''}}" :class="'nav-item btn confirmbtn ' + (canBuy!=''?'disabled':'')">{{canBuy==''?'确定':'库存不足'}}</a>
```
需手动将tap里面的内容放置在一个方法里面



## getCurrentPages
```
const pages = getCurrentPages();
let currentPage = pages[pages.length-1];
let options =currentPage.options || currentPage.$route.query;
```


## SyntaxError: Void elements do not have end tags "input" (83:149)
input没有结束标签，然后格式化插件报错，编译不报错就行。

## 在vue初始化前就调用了getApp()
多发生于在main里加载的组件里调用了getApp();


## 文件查找失败：'./iconfont.eot?t=1583910898195'
转换后运行时，控制台报如下错误：
```
11:31:22.930 文件查找失败：'./iconfont.eot?t=1583910898195' at App.vue:4
11:31:22.932 文件查找失败：'./iconfont.svg?t=1583910898195' at App.vue:7
11:31:22.939 文件查找失败：'./iconfont.ttf?t=1583910898195' at App.vue:6
11:31:22.939 文件查找失败：'./iconfont.woff?t=1583910898195' at App.vue:5
```
本来，想着在工具里能把这个错误给规避掉，后面发现异常情况太多，没啥太好的办法。

原因： 
在微信小程序里，wxss里面引用的路径，如果不存在是不会报错的。
但在uniapp里就不一样了，路径不存在会在编译阶段直接报错。

根据报错信息，显然是说字体文件路径不存在，原因有二，要么是文件路径不对，要么是对应文件不存在。

在这里，文件不存在的可能性更高一些！


#### 解决方案
一般来说，这种情况，都是引用了字体图标，而没有把字体文件一并导入导致的。
因此找到对应的css文件，然后将里面的字体路径引用注释或删除，保留base64的部分即可。


#### 重现示例
```
//iconfont.css
@font-face {
  font-family: "iconfont";
  src: url('iconfont.eot?t=1583910898195'); /* IE9 */
  src: url('iconfont.eot?t=1583910898195#iefix') format('embedded-opentype'), /* IE6-IE8 */
  url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAF+sAAsAAAAAwAgAAF9YAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHE..........(大段内容省略).........twPFbcAAAAA') format('woff2'),
  url('iconfont.woff?t=1583910898195') format('woff'),
  url('iconfont.ttf?t=1583910898195') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
  url('iconfont.svg?t=1583910898195#iconfont') format('svg'); /* iOS 4.1- */
}
```

#### 修复示例 
```
//iconfont.css
@font-face {
  font-family: "iconfont";
  src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAF+sAAsAAAAAwAgAAF9YAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHE..........(大段内容省略).........twPFbcAAAAA') format('woff2');
}
```
修改时，请仔细对比这两段代码的差异。



## 转换使用weui的小程序，运行时报错"ReferenceError: moduleId is not defined"
原因是工具对weui库有点不太适应，weui看起来是webpack直接打包出来的，然而里面又有小程序的代码。

请在转换之前将代码里webpack结构删除（这不会影响小程序运行的）， 保留小程序代码（见下方示例）。

然后！然后再进行转换为uniapp项目即可！

！！！
如果有weui项目，请先复制一份小程序项目！！！记住先复制一份，然后使用群文件里面的weui进行替换后，再对复制后的这份小程序项目进行转换！！！
！！！

#### 重现示例
```
//cell.js
module.exports =
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 9);
/******/ })
/************************************************************************/
/******/ ({

/***/ 9:
/***/ (function(module, exports, __webpack_require__) {

"use strict";

/**我是注释：下面是小程序的代码，是要保留的代码*/
Component({
  options: {
    addGlobalClass: true,
    multipleSlots: true
  },
  properties: {
    hover: {
      type: Boolean,
      value: false
    },
    link: {
      type: Boolean,
      value: false
    }
  },
  relations: {
    '../form/form': {
      type: 'ancestor'
    },
    '../cells/cells': {
      type: 'ancestor'
    }
  },
  methods: {

  }
});
/**我是注释：上面是小程序的代码，是要保留的代码*/

/***/ })

/******/ });
```

#### 修复示例 
将上述代码里面的“要保留的代码”保留，其他代码都删除，然后重新进行转换即可（可解决部分问题，如需完全解决，请见上面方案）。   
注：其他文件有如此情形也应一并修改。   
修改后的代码如下：   

```
//cell.js
Component({
  options: {
    addGlobalClass: true,
    multipleSlots: true
  },
  properties: {
    hover: {
      type: Boolean,
      value: false
    },
    link: {
      type: Boolean,
      value: false
    }
  },
  relations: {
    '../form/form': {
      type: 'ancestor'
    },
    '../cells/cells': {
      type: 'ancestor'
    }
  },
  methods: {

  }
});
```