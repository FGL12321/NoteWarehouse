[TOC]



# VUE入门

## 1，VUE

### 1.1  概述

接下来我们学习一款前端的框架，就是 VUE。

==Vue 是一套前端框架，免除原生JavaScript中的DOM操作，简化书写。==

我们之前也学习过后端的框架 `Mybatis` ，`Mybatis` 是用来简化 `jdbc` 代码编写的；而 `VUE` 是前端的框架，是用来简化 `JavaScript` 代码编写的。前一天我们做了一个综合性的案例，里面进行了大量的DOM操作，如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831112115508.png" alt="image-20210831112115508" style="zoom:70%;" />

学习了 `VUE` 后，这部分代码我们就不需要再写了。那么 `VUE` 是如何简化 DOM 书写呢？

==基于MVVM(Model-View-ViewModel)思想，实现数据的双向绑定，将编程的关注点放在数据上。==之前我们是将关注点放在了 DOM 操作上；而要了解 `MVVM` 思想，必须先聊聊 `MVC` 思想，如下图就是 `MVC` 思想图解

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831113940588.png" alt="image-20210831113940588" style="zoom:70%;" />

C 就是咱们 js 代码，M 就是数据，而 V 是页面上展示的内容，如下图是我们之前写的代码

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831114227585.png" alt="image-20210831114227585" style="zoom:70%;" />

`MVC` 思想是没法进行双向绑定的。双向绑定是指当数据模型数据发生变化时，页面展示的会随之发生变化，而如果表单数据发生变化，绑定的模型数据也随之发生变化。接下来我们聊聊 `MVVM` 思想，如下图是三个组件图解

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831114805052.png" alt="image-20210831114805052" style="zoom:80%;" />

图中的 `Model` 就是我们的数据，`View` 是视图，也就是页面标签，用户可以通过浏览器看到的内容；`Model` 和 `View` 是通过 `ViewModel` 对象进行双向绑定的，而 `ViewModel` 对象是 `Vue` 提供的。接下来让大家看一下双向绑定的效果，下图是提前准备的代码，输入框绑定了 `username` 模型数据，而在页面上也使用 `{{}}` 绑定了 `username` 模型数据

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831115645528.png" alt="image-20210831115645528" style="zoom:70%;" />

通过浏览器打开该页面可以看到如下页面

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831115902537.png" alt="image-20210831115902537" style="zoom:80%;" />

当我们在输入框中输入内容，而输入框后面随之实时的展示我们输入的内容，这就是双向绑定的效果。

### 1.2  快速入门

Vue 使用起来是比较简单的，总共分为如下三步：

1. **新建 HTML 页面，引入 Vue.js文件**

   ```html
   <script src="js/vue.js"></script>
   ```

2. **在JS代码区域，创建Vue核心对象，进行数据绑定**

   ```js
   new Vue({
       el: "#app",
       data() {
           return {
               username: ""
           }
       }
   });
   ```

   创建 Vue 对象时，需要传递一个 js 对象，而该对象中需要如下属性：

   * `el` ： 用来指定哪儿些标签受 Vue 管理。 该属性取值 `#app` 中的 `app` 需要是受管理的标签的id属性值
   * `data` ：用来定义数据模型
   * `methods` ：用来定义函数。这个我们在后面就会用到

3. **编写视图**

   ```html
   <div id="app">
       <input name="username" v-model="username" >
       {{username}}
   </div>
   ```

   `{{}}` 是 Vue 中定义的 `插值表达式` ，在里面写数据模型，到时候会将该模型的数据值展示在这个位置。

**整体代码如下：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <input v-model="username">
    <!--插值表达式-->
    {{username}}
</div>
<script src="js/vue.js"></script>
<script>
    //1. 创建Vue核心对象
    new Vue({
        el:"#app",
        data(){  // data() 是 ECMAScript 6 版本的新的写法
            return {
                username:""
            }
        }

        /*data: function () {
            return {
                username:""
            }
        }*/
    });

</script>
</body>
</html>
```

### 1.3  Vue 指令

**指令：**HTML 标签上带有 v- 前缀的特殊属性，不同指令具有不同含义。例如：v-if，v-for…

常用的指令有：

| **指令**  | **作用**                                            |
| --------- | --------------------------------------------------- |
| v-bind    | 为HTML标签绑定属性值，如设置  href , css样式等      |
| v-model   | 在表单元素上创建双向数据绑定                        |
| v-on      | 为HTML标签绑定事件                                  |
| v-if      | 条件性的渲染某元素，判定为true时渲染,否则不渲染     |
| v-else    |                                                     |
| v-else-if |                                                     |
| v-show    | 根据条件展示某元素，区别在于切换的是display属性的值 |
| v-for     | 列表渲染，遍历容器的元素或者对象的属性              |

接下来我们挨个学习这些指令

#### 1.3.1  v-bind & v-model 指令

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831150101736.png" alt="image-20210831150101736" style="zoom:70%;" />

* **v-bind**

  该指令可以给标签原有属性绑定模型数据。这样模型数据发生变化，标签属性值也随之发生变化

  例如：

  ```html
  <a v-bind:href="url">百度一下</a>
  ```

  上面的 `v-bind:"`  可以简化写成 `:`  ，如下：

  ```html
  <!--
  	v-bind 可以省略
  -->
  <a :href="url">百度一下</a>
  ```

* **v-model**

  该指令可以给表单项标签绑定模型数据。这样就能实现双向绑定效果。例如：

  ```html
  <input name="username" v-model="username">
  ```

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <a v-bind:href="url">点击一下</a>
    <a :href="url">点击一下</a>
    <input v-model="url">
</div>

<script src="js/vue.js"></script>
<script>
    //1. 创建Vue核心对象
    new Vue({
        el:"#app",
        data(){
            return {
                username:"",
                url:"https://www.baidu.com"
            }
        }
    });
</script>
</body>
</html>
```

通过浏览器打开上面页面，并且使用检查查看超链接的路径，该路径会根据输入框输入的路径变化而变化，这是因为超链接和输入框绑定的是同一个模型数据

![image-20230219131932684](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230219131932684.png)

#### 1.3.2  v-on 指令

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831151231955.png" alt="image-20210831151231955" style="zoom:70%;" />

我们在页面定义一个按钮，并给该按钮使用 `v-on` 指令绑定单击事件，html代码如下

```html
<input type="button" value="一个按钮" v-on:click="show()">
```

而使用 `v-on` 时还可以使用简化的写法，将 `v-on:` 替换成 `@`，html代码如下

```html
<input type="button" value="一个按钮" @click="show()">
```

上面代码绑定的 `show()` 需要在 Vue 对象中的 `methods` 属性中定义出来

```js
new Vue({
    el: "#app",
    methods: {
        show(){
            alert("我被点了");
        }
    }
});
```

> ==注意：`v-on:` 后面的事件名称是之前原生事件属性名去掉on。==
>
> 例如：
>
> * 单击事件 ： 事件属性名是 onclick，而在vue中使用是 `v-on:click`
> * 失去焦点事件：事件属性名是 onblur，而在vue中使用时 `v-on:blur`

**整体页面代码如下：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <input type="button" value="一个按钮" v-on:click="show()"><br>
    <input type="button" value="一个按钮" @click="show()">
</div>
<script src="js/vue.js"></script>
<script>
    //1. 创建Vue核心对象
    new Vue({
        el:"#app",
        data(){
            return {
                username:"",
            }
        },
        methods:{
            show(){
                alert("我被点了...");
            }
        }
    });
</script>
</body>
</html>
```

#### 1.3.3  条件判断指令

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831151904081.png" alt="image-20210831151904081" style="zoom:70%;" />

接下来通过代码演示一下。在 Vue中定义一个 `count` 的数据模型，如下

```js
//1. 创建Vue核心对象
new Vue({
    el:"#app",
    data(){
        return {
            count:3
        }
    }
});
```

现在要实现，当 `count` 模型的数据是3时，在页面上展示 `div1` 内容；当 `count` 模型的数据是4时，在页面上展示 `div2` 内容；`count` 模型数据是其他值时，在页面上展示 `div3`。这里为了动态改变模型数据 `count` 的值，再定义一个输入框绑定 `count` 模型数据。html 代码如下：

```html
<div id="app">
    <div v-if="count == 3">div1</div>
    <div v-else-if="count == 4">div2</div>
    <div v-else>div3</div>
    <hr>
    <input v-model="count">
</div>
```

**整体页面代码如下：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <div v-if="count == 3">div1</div>
    <div v-else-if="count == 4">div2</div>
    <div v-else>div3</div>
    <hr>
    <input v-model="count">
</div>

<script src="js/vue.js"></script>
<script>
    //1. 创建Vue核心对象
    new Vue({
        el:"#app",
        data(){
            return {
                count:3
            }
        }
    });
</script>
</body>
</html>
```

通过浏览器打开页面并在输入框输入不同的值，效果如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831154300325.png" alt="image-20210831154300325" style="zoom:80%;" />

然后我们在看看 `v-show` 指令的效果，如果模型数据 `count ` 的值是3时，展示 `div v-show` 内容，否则不展示，html页面代码如下

```html
<div v-show="count == 3">div v-show</div>
<br>
<input v-model="count">
```

浏览器打开效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831154547780.png" alt="image-20210831154547780" style="zoom:80%;" />

通过上面的演示，发现 `v-show` 和 `v-if` 效果一样，那它们到底有什么区别呢？我们根据浏览器的检查功能查看源代码

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831154759672.png" alt="image-20210831154759672" style="zoom:80%;" />

通过上图可以看出 `v-show` 不展示的原理是给对应的标签添加 `display` css属性，并将该属性值设置为 `none` ，这样就达到了隐藏的效果。而 `v-if` 指令是条件不满足时根本就不会渲染。

#### 1.3.4  v-for 指令

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831155204829.png" alt="image-20210831155204829" style="zoom:80%;" />

这个指令看到名字就知道是用来遍历的，该指令使用的格式如下：

```html
<标签 v-for="变量名 in 集合模型数据">
    {{变量名}}
</标签>
```

> ==注意：需要循环那个标签，`v-for` 指令就写在那个标签上。==

如果在页面需要使用到集合模型数据的索引，就需要使用如下格式：

```html
<标签 v-for="(变量名,索引变量) in 集合模型数据">
    <!--索引变量是从0开始，所以要表示序号的话，需要手动的加1-->
   {{索引变量 + 1}} {{变量名}}
</标签>
```

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <div v-for="addr in addrs">
        {{addr}} <br>
    </div>

    <hr>
    <div v-for="(addr,i) in addrs">
        {{i+1}}--{{addr}} <br>
    </div>
</div>

<script src="js/vue.js"></script>
<script>

    //1. 创建Vue核心对象
    new Vue({
        el:"#app",
        data(){
            return {
                addrs:["北京","上海","西安"]
            }
        }
    });
</script>
</body>
</html>
```

通过浏览器打开效果如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831155837801.png" alt="image-20210831155837801" style="zoom:80%;" />

### 1.4  生命周期 

生命周期的八个阶段：每触发一个生命周期事件，会自动执行一个生命周期方法，这些生命周期方法也被称为钩子方法。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831160239294.png" alt="image-20210831160239294" style="zoom:80%;" />

下图是 Vue 官网提供的从创建 Vue 到效果 Vue 对象的整个过程及各个阶段对应的钩子函数

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831160335496.png" alt="image-20210831160335496" style="zoom:80%;" />

看到上面的图，大家无需过多的关注这张图。这些钩子方法我们只关注 `mounted` 就行了。

`mounted`：挂载完成，Vue初始化成功，HTML页面渲染成功。而以后我们会在该方法中==发送异步请求，加载数据。==

### 1.5  案例

#### 1.5.1  需求

使用 Vue 简化我们在前一天ajax学完后做的品牌列表数据查询和添加功能

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831161040800.png" alt="image-20210831161040800" style="zoom:80%;" />

此案例只是使用 Vue 对前端代码进行优化，后端代码无需修改。

#### 1.5.2  查询所有功能

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831161346678.png" alt="image-20210831161346678" style="zoom:80%;" />

1. **在 brand.html 页面引入 vue 的js文件**

   ```html
   <script src="js/vue.js"></script>
   ```

2. **创建 Vue 对象**

   * 在 Vue 对象中定义模型数据
   * 在钩子函数中发送异步请求，并将响应的数据赋值给数据模型

   ```js
   new Vue({
       el: "#app",
       data(){
           return{
               brands:[]
           }
       },
       mounted(){
           // 页面加载完成后，发送异步请求，查询数据
           var _this = this;
           axios({
               method:"get",
               url:"http://localhost:8080/brand-demo/selectAllServlet"
           }).then(function (resp) {
               _this.brands = resp.data;
           })
       }
   })
   ```

3. **修改视图**

   * 定义 `<div id="app"></div>` ，指定该 `div` 标签受 Vue 管理

   * 将 `body` 标签中所有的内容拷贝作为上面 `div` 标签中

   * 删除表格的多余数据行，只留下一个

   * 在表格中的数据行上使用 `v-for` 指令遍历

     ```html
     <tr v-for="(brand,i) in brands" align="center">
         <td>{{i + 1}}</td>
         <td>{{brand.brandName}}</td>
         <td>{{brand.companyName}}</td>
         <td>{{brand.ordered}}</td>
         <td>{{brand.description}}</td>
         <td>{{brand.statusStr}}</td>
         <td><a href="#">修改</a> <a href="#">删除</a></td>
     </tr>
     ```

**整体页面代码如下：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <a href="addBrand.html"><input type="button" value="新增"></a><br>
    <hr>
    <table id="brandTable" border="1" cellspacing="0" width="100%">
        <tr>
            <th>序号</th>
            <th>品牌名称</th>
            <th>企业名称</th>
            <th>排序</th>
            <th>品牌介绍</th>
            <th>状态</th>
            <th>操作</th>
        </tr>
        <!--
            使用v-for遍历tr
        -->
        <tr v-for="(brand,i) in brands" align="center">
            <td>{{i + 1}}</td>
            <td>{{brand.brandName}}</td>
            <td>{{brand.companyName}}</td>
            <td>{{brand.ordered}}</td>
            <td>{{brand.description}}</td>
            <td>{{brand.statusStr}}</td>
            <td><a href="#">修改</a> <a href="#">删除</a></td>
        </tr>
    </table>
</div>
<script src="js/axios-0.18.0.js"></script>
<script src="js/vue.js"></script>

<script>
    new Vue({
        el: "#app",
        data(){
            return{
                brands:[]
            }
        },
        mounted(){
            // 页面加载完成后，发送异步请求，查询数据
            var _this = this;
            axios({
                method:"get",
                url:"http://localhost:8080/brand-demo/selectAllServlet"
            }).then(function (resp) {
                _this.brands = resp.data;
            })
        }
    })
</script>
</body>
</html>
```

#### 1.5.3  添加功能

页面操作效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831163001830.png" alt="image-20210831163001830" style="zoom:80%;" />

整体流程如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831163035298.png" alt="image-20210831163035298" style="zoom:70%;" />

> ==注意：前端代码的关键点在于使用 `v-model` 指令给标签项绑定模型数据，利用双向绑定特性，在发送异步请求时提交数据。==

1. **在 addBrand.html 页面引入 vue 的js文件**

   ```html
   <script src="js/vue.js"></script>
   ```

2. **创建 Vue 对象**

   * 在 Vue 对象中定义模型数据 `brand` 
   * 定义一个 `submitForm()` 函数，用于给 `提交` 按钮提供绑定的函数
   * 在 `submitForm()` 函数中发送 ajax 请求，并将模型数据 `brand` 作为参数进行传递

   ```js
   new Vue({
       el: "#app",
       data(){
           return {
               brand:{}
           }
       },
       methods:{
           submitForm(){
               // 发送ajax请求，添加
               var _this = this;
               axios({
                   method:"post",
                   url:"http://localhost:8080/brand-demo/addServlet",
                   data:_this.brand
               }).then(function (resp) {
                   // 判断响应数据是否为 success
                   if(resp.data == "success"){
                       location.href = "http://localhost:8080/brand-demo/brand.html";
                   }
               })
   
           }
       }
   })
   ```

3. **修改视图**

   * 定义 `<div id="app"></div>` ，指定该 `div` 标签受 Vue 管理

   * 将 `body` 标签中所有的内容拷贝作为上面 `div` 标签中

   * 给每一个表单项标签绑定模型数据。最后这些数据要被封装到 `brand` 对象中

     ```html
     <div id="app">
         <h3>添加品牌</h3>
         <form action="" method="post">
             品牌名称：<input id="brandName" v-model="brand.brandName" name="brandName"><br>
             企业名称：<input id="companyName" v-model="brand.companyName" name="companyName"><br>
             排序：<input id="ordered" v-model="brand.ordered" name="ordered"><br>
             描述信息：<textarea rows="5" cols="20" id="description" v-model="brand.description" name="description"></textarea><br>
             状态：
             <input type="radio" name="status" v-model="brand.status" value="0">禁用
             <input type="radio" name="status" v-model="brand.status" value="1">启用<br>
     
             <input type="button" id="btn" @click="submitForm" value="提交">
         </form>
     </div>
     ```

**整体页面代码如下：**

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>添加品牌</title>
</head>
<body>
<div id="app">
    <h3>添加品牌</h3>
    <form action="" method="post">
        品牌名称：<input id="brandName" v-model="brand.brandName" name="brandName"><br>
        企业名称：<input id="companyName" v-model="brand.companyName" name="companyName"><br>
        排序：<input id="ordered" v-model="brand.ordered" name="ordered"><br>
        描述信息：<textarea rows="5" cols="20" id="description" v-model="brand.description" name="description"></textarea><br>
        状态：
        <input type="radio" name="status" v-model="brand.status" value="0">禁用
        <input type="radio" name="status" v-model="brand.status" value="1">启用<br>

        <input type="button" id="btn" @click="submitForm" value="提交">
    </form>
</div>
<script src="js/axios-0.18.0.js"></script>
<script src="js/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data(){
            return {
                brand:{}
            }
        },
        methods:{
            submitForm(){
                // 发送ajax请求，添加
                var _this = this;
                axios({
                    method:"post",
                    url:"http://localhost:8080/brand-demo/addServlet",
                    data:_this.brand
                }).then(function (resp) {
                    // 判断响应数据是否为 success
                    if(resp.data == "success"){
                        location.href = "http://localhost:8080/brand-demo/brand.html";
                    }
                })
            }
        }
    })
</script>
</body>
</html>
```

通过上面的优化，前端代码确实简化了不少。但是页面依旧是不怎么好看，那么接下来我们学习 Element，它可以美化页面。