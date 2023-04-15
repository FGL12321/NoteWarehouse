

[TOC]

# 十.JQuery

# 1.JQuery快速入门

## 1.1、JQuery介绍

- jQuery 是一个 JavaScript 库。
- 所谓的库，就是一个 JS 文件，里面封装了很多预定义的函数，比如获取元素，执行隐藏、移动等，目的就是在使用时直接调用，不需要再重复定义，这样就可以极大地简化了 JavaScript 编程。
- jQuery 官网：https://www.jquery.com

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/JQuery%E4%BB%8B%E7%BB%8D.png)

## 1.2、JQuery快速入门

### **开发思路**

1. 编写 HTML 文档。 
2. 引入 jQuery 文件。
3. 使用 jQuery 获取元素。
4. 使用浏览器测试。

### **代码实现**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>快速入门</title>
</head>
<body>
    <div id="div">我是div</div>
</body>
<!--引入 jQuery 文件-->
<script src="js/jquery-3.3.1.min.js"></script>
<script>
    // JS方式，通过id属性值来获取div元素
    let jsDiv = document.getElementById("div");
    //alert(jsDiv);
    //alert(jsDiv.innerHTML);

    // jQuery方式，通过id属性值来获取div元素
    let jqDiv = $("#div");
    alert(jqDiv);
    alert(jqDiv.html());
</script>
</html>
```

## 1.3、小结

- jQuery 是一个 JavaScript 库。
- 说白了就是定义好的一个 JS 文件，内部封装了很多功能，可以大大简化我们的 JS 操作步骤。
- jQuery 官网：https://www.jquery.com。
- 要想使用，必须要引入该文件。
- jQuery 的核心语法 $();

# 2、JQuery基本语法

## 2.1、JS对象和JQuery对象转换

- jQuery 本质上虽然也是 JS，但如果想使用 jQuery 的属性和方法那么必须保证对象是 jQuery 对象，而不是 JS 方式获得的 DOM 对象，二者的 API 方法不能混合使用，若想使用对方的 API，需要进行对象的转换。

- **JS 的 DOM 对象转换成 jQuery 对象**

  ```js
  //$(JS 的 DOM 对象);
  
  // JS方式，通过id属性值获取div元素
  let jsDiv = document.getElementById("div");
  alert(jsDiv.innerHTML);
  //alert(jsDiv.html());  JS对象无法使用jQuery里面的功能
  
  // 将 JS 对象转换为jQuery对象
  let jq = $(jsDiv);
  alert(jq.html());
  ```

- **jQuery 对象转换成 JS 对象**

  ```js
  /*jQuery 对象[索引];
  jQuery 对象.get(索引);*/
  
  // jQuery方式，通过id属性值获取div元素
  let jqDiv = $("#div");
  alert(jqDiv.html());
  // alert(jqDiv.innerHTML);   jQuery对象无法使用JS里面的功能
  
  // 将 jQuery对象转换为JS对象
  let js = jqDiv[0];
  alert(js.innerHTML);
  ```

## 2.2、事件的基本使用

- **常用的事件**

  ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E5%B8%B8%E7%94%A8%E4%BA%8B%E4%BB%B6.png)

- **在 jQuery 中将事件封装成了对应的方法。去掉了 JS 中的 .on 语法。**

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>事件的使用</title>
  </head>
  <body>
      <input type="button" id="btn" value="点我">
      <br>
      <input type="text" id="input">
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      //单击事件
      $("#btn").click(function(){
          alert("点我干嘛?");
      });
  
      //获取焦点事件
      // $("#input").focus(function(){
      //     alert("你要输入数据啦...");
      // });
  
      //失去焦点事件
      $("#input").blur(function(){
          alert("你输入完成啦...");
      });
  </script>
  </html>
  ```

## 2.3、事件的绑定和解绑

- **绑定事件**

  **//jQuery 对象.on(事件名称,执行的功能);**

  ```js
  //给btn1按钮绑定单击事件
  $("#btn1").on("click",function(){
  	alert("点我干嘛?");
  });
  ```

- **解绑事件**

  如果不指定事件名称，则会把该对象绑定的所有事件都解绑

  **//jQuery 对象.off(事件名称);**

  ```js
  //通过btn2解绑btn1的单击事件
  $("#btn2").on("click",function(){
  	$("#btn1").off("click");
  });
  ```

## 2.4、事件的切换

事件的切换：需要给同一个对象绑定多个事件，而且多个事件还有先后顺序关系。

- **方式一：单独定义**

  $(元素).事件方法名1(要执行的功能); 

  $(元素).事件方法名2(要执行的功能);

  ```js
  //方式一 单独定义
  $("#div").mouseover(function () {
      //背景色：红色
      //$("#div").css("background","red");
      $(this).css("background", "red");
  });
  $("#div").mouseout(function () {
      //背景色：蓝色
      //$("#div").css("background","blue");
      $(this).css("background", "blue");
  });
  ```

- **方式二：链式定义**

  $(元素).事件方法名1(要执行的功能) 

  .事件方法名2(要执行的功能);

  ```js
  //方式二 链式定义
  $("#div").mouseover(function(){
     $(this).css("background","red");
  }).mouseout(function(){
     $(this).css("background","blue");
  });
  ```

## 2.5、遍历操作

- **方式一：传统方式**

  ```js
  for(let i = 0; i < 容器对象长度; i++){
  	执行功能;
  }
  ```

  ```js
  //方式一：传统方式
  $("#btn").click(function(){
      let lis = $("li");
      for(let i = 0 ; i < lis.length; i++) {
          alert(i + ":" + lis[i].innerHTML);
      }
  });
  ```

- **方式二：对象.each()方法**

  ```js
  容器对象.each(function(index,ele){
  	执行功能;
  });
  ```

  ```js
  //方式二：对象.each()方法
  $("#btn").click(function(){
      let lis = $("li");
      lis.each(function(index,ele){
          alert(index + ":" + ele.innerHTML);
      });
  });
  ```

- **方式三：$.each()方法**

  ```tex
  $.each(容器对象,function(index,ele){
  	执行功能;
  });
  ```

  ```js
  //方式三：$.each()方法
  $("#btn").click(function(){
      let lis = $("li");
      $.each(lis,function(index,ele){
          alert(index + ":" + ele.innerHTML);
      });
  });
  ```

- **方式四：for of语句**

  ```tex
  for(ele of 容器对象){
  	执行功能;
  }
  ```

  ```js
  //方式四：for of 语句遍历
  $("#btn").click(function(){
      let lis = $("li");
      for(ele of lis){
          alert(ele.innerHTML);
      }
  });
  ```

## 2.6、小结

- **JS 对象和 jQuery 对象相互转换**
  - $(JS 的 DOM 对象)：将 JS 对象转为 jQuery 对象。
  - jQuery 对象[索引] jQuery
  - 对象.get(索引)：将 jQuery 对象转为 JS 对象。
- **事件**
  - 在 jQuery 中将事件封装成了对应的方法。去掉了 JS 中的 .on 语法。
  - on(事件名称,执行的功能)：绑定事件。
  - off(事件名称)：解绑事件。
- **遍历**
  - 传统方式。
  - 对象.each() 方法。
  - $.each() 方法。
  - for of 语句。

# 3、JQuery选择器

## 3.1、基本选择器

- 选择器：类似于 CSS 的选择器，可以帮助我们获取元素。

- 例如：id 选择器、类选择器、元素选择器、属性选择器等等。

- jQuery 中选择器的语法：$();

  

  ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E5%9F%BA%E6%9C%AC%E9%80%89%E6%8B%A9%E5%99%A8.png)

  **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>基本选择器</title>
  </head>
  <body>
      <div id="div1">div1</div>
      <div class="cls">div2</div>
      <div class="cls">div3</div>
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      //1.元素选择器   $("元素的名称")
      let divs = $("div");
      //alert(divs.length);
  
      //2.id选择器    $("#id的属性值")
      let div1 = $("#div1");
      //alert(div1);
  
      //3.类选择器     $(".class的属性值")
      let cls = $(".cls");
      alert(cls.length);
  
  </script>
  </html>
  ```

## 3.2、层级选择器

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E5%B1%82%E7%BA%A7%E9%80%89%E6%8B%A9%E5%99%A8.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>层级选择器</title>
  </head>
  <body>
      <div>
          <span>s1
              <span>s1-1</span>
              <span>s1-2</span>
          </span>
          <span>s2</span>
      </div>
  
      <div></div>
      <p>p1</p>
      <p>p2</p>
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      // 1. 后代选择器 $("A B");      A下的所有B(包括B的子级)
      let spans1 = $("div span");
      //alert(spans1.length);
  
      // 2. 子选择器   $("A > B");    A下的所有B(不包括B的子级)
      let spans2 = $("div > span");
      //alert(spans2.length);
  
      // 3. 兄弟选择器 $("A + B");    A相邻的下一个B
      let ps1 = $("div + p");
      //alert(ps1.length);
  
      // 4. 兄弟选择器 $("A ~ B");    A相邻的所有B
      let ps2 = $("div ~ p");
      alert(ps2.length);
      
  </script>
  </html>
  ```

## 3.3、属性选择器

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E5%B1%9E%E6%80%A7%E9%80%89%E6%8B%A9%E5%99%A8.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>属性选择器</title>
  </head>
  <body>
      <input type="text">
      <input type="password">
      <input type="password">
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      //1.属性名选择器   $("元素[属性名]")
      let in1 = $("input[type]");
      //alert(in1.length);
  
  
      //2.属性值选择器   $("元素[属性名=属性值]")
      let in2 = $("input[type='password']");
      alert(in2.length);
  
  </script>
  </html>
  ```

## 3.4、过滤器选择器

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E8%BF%87%E6%BB%A4%E5%99%A8%E9%80%89%E6%8B%A9%E5%99%A8.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>过滤器选择器</title>
  </head>
  <body>
      <div>div1</div>
      <div id="div2">div2</div>
      <div>div3</div>
      <div>div4</div>
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      // 1.首元素选择器	$("A:first");
      let div1 = $("div:first");
      //alert(div1.html());
  
      // 2.尾元素选择器	$("A:last");
      let div4 = $("div:last");
      //alert(div4.html());
  
      // 3.非元素选择器	$("A:not(B)");
      let divs1 = $("div:not(#div2)");
      //alert(divs1.length);
  
      // 4.偶数选择器	    $("A:even");
      let divs2 = $("div:even");
      //alert(divs2.length);
      //alert(divs2[0].innerHTML);
      //alert(divs2[1].innerHTML);
  
      // 5.奇数选择器	    $("A:odd");
      let divs3 = $("div:odd");
      //alert(divs3.length);
      //alert(divs3[0].innerHTML);
      //alert(divs3[1].innerHTML);
  
      // 6.等于索引选择器	 $("A:eq(index)");
      let div3 = $("div:eq(2)");
      //alert(div3.html());
  
      // 7.大于索引选择器	 $("A:gt(index)");
      let divs4 = $("div:gt(1)");
      //alert(divs4.length);
      //alert(divs4[0].innerHTML);
      //alert(divs4[1].innerHTML);
  
      // 8.小于索引选择器	 $("A:lt(index)");
      let divs5 = $("div:lt(2)");
      alert(divs5.length);
      alert(divs5[0].innerHTML);
      alert(divs5[1].innerHTML);
      
  </script>
  </html>
  ```

## 3.5、表单属性选择器

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E8%A1%A8%E5%8D%95%E5%B1%9E%E6%80%A7%E9%80%89%E6%8B%A9%E5%99%A8.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>表单属性选择器</title>
  </head>
  <body>
      <input type="text" disabled>
      <input type="text" >
      <input type="radio" name="gender" value="men" checked>男
      <input type="radio" name="gender" value="women">女
      <input type="checkbox" name="hobby" value="study" checked>学习
      <input type="checkbox" name="hobby" value="sleep" checked>睡觉
      <select>
          <option>---请选择---</option>
          <option selected>本科</option>
          <option>专科</option>
      </select>
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      // 1.可用元素选择器  $("A:enabled");
      let ins1 = $("input:enabled");
      //alert(ins1.length);
  
      // 2.不可用元素选择器  $("A:disabled");
      let ins2 = $("input:disabled");
      //alert(ins2.length);
  
      // 3.单选/复选框被选中的元素  $("A:checked");
      let ins3 = $("input:checked");
      //alert(ins3.length);
      //alert(ins3[0].value);
      //alert(ins3[1].value);
      //alert(ins3[2].value);
  
      // 4.下拉框被选中的元素   $("A:selected");
      let select = $("select option:selected");
      alert(select.html());
      
  </script>
  </html>
  ```

## 3.6、小结

- 选择器：类似于 CSS 的选择器，可以帮助我们获取元素。
- jQuery 中选择器的语法：$(); 
- 基本选择器
  - $("元素的名称");
  - $("#id的属性值");
  - $(".class的属性值");
- 层级选择器
  - $("A B");
  - $("A > B");
- 属性选择器
  - $("A[属性名]");
  - $("A[属性名=属性值]");
- 过滤器选择器
  - $("A:even");
  - $("A:odd");
- 表单属性选择器
  - $("A:disabled");
  - $("A:checked");
  - $("A:selected");

# 4、JQuery DOM

## 4.1、操作文本

- **常用方法**

  ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E6%93%8D%E4%BD%9C%E6%96%87%E6%9C%AC.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>操作文本</title>
  </head>
  <body>
      <div id="div">我是div</div>
      <input type="button" id="btn1" value="获取div的文本">
      <input type="button" id="btn2" value="设置div的文本">
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
       //1. html()   获取标签的文本内容
       $("#btn1").click(function(){
           //获取div标签的文本内容
           let value = $("#div").html();
           alert(value);
       });
  
       //2. html(value)   设置标签的文本内容，解析标签
       $("#btn2").click(function(){
           //设置div标签的文本内容
           //$("#div").html("我真的是div");
           $("#div").html("<b>我真的是div</b>");
       });
  </script>
  </html>
  ```

## 4.2、操作对象

- **常用方法**

  ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E6%93%8D%E4%BD%9C%E5%AF%B9%E8%B1%A1.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>操作对象</title>
  </head>
  <body>
      <div id="div"></div>
      <input type="button" id="btn1" value="添加一个span到div"> <br><br><br>
  
      <input type="button" id="btn2" value="将加油添加到城市列表最下方"> &nbsp;&nbsp;&nbsp;
      <input type="button" id="btn3" value="将加油添加到城市列表最上方"> &nbsp;&nbsp;&nbsp;
      <input type="button" id="btn4" value="将雄起添加到上海下方"> &nbsp;&nbsp;&nbsp;
      <input type="button" id="btn5" value="将雄起添加到上海上方"> &nbsp;&nbsp;&nbsp;
      <ul id="city">
          <li id="bj">北京</li>
          <li id="sh">上海</li>
          <li id="gz">广州</li>
          <li id="sz">深圳</li>
      </ul>
      <ul id="desc">
          <li id="jy">加油</li>
          <li id="xq">雄起</li>
      </ul>  <br><br><br>
      <input type="button" id="btn6" value="将雄起删除"> &nbsp;&nbsp;&nbsp;
      <input type="button" id="btn7" value="将描述列表全部删除"> &nbsp;&nbsp;&nbsp;
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      /*
          1. $("元素")   创建指定元素
          2. append(element)   添加成最后一个子元素，由添加者对象调用
          3. appendTo(element) 添加成最后一个子元素，由被添加者对象调用
          4. prepend(element)  添加成第一个子元素，由添加者对象调用
          5. prependTo(element) 添加成第一个子元素，由被添加者对象调用
          6. before(element)    添加到当前元素的前面，两者之间是兄弟关系，由添加者对象调用
          7. after(element)     添加到当前元素的后面，两者之间是兄弟关系，由添加者对象调用
          8. remove()           删除指定元素(自己移除自己)
          9. empty()            清空指定元素的所有子元素
      */
      
      // 按钮一：添加一个span到div
      $("#btn1").click(function(){
          let span = $("<span>span</span>");
          $("#div").append(span);
      });
      
  
      //按钮二：将加油添加到城市列表最下方
      $("#btn2").click(function(){
          //$("#city").append($("#jy"));
          $("#jy").appendTo($("#city"));
      });
  
      //按钮三：将加油添加到城市列表最上方
      $("#btn3").click(function(){
          //$("#city").prepend($("#jy"));
          $("#jy").prependTo($("#city"));
      });
      
  
      //按钮四：将雄起添加到上海下方
      $("#btn4").click(function(){
          $("#sh").after($("#xq"));
      });
      
  
      //按钮五：将雄起添加到上海上方
      $("#btn5").click(function(){
          $("#sh").before($("#xq"));
      });
  
      //按钮六：将雄起删除
      $("#btn6").click(function(){
          $("#xq").remove();
      });
      
  
      //按钮七：将描述列表全部删除
      $("#btn7").click(function(){
          $("#desc").empty();
      });
      
  </script>
  </html>
  ```

## 4.3、操作样式

- **常用方法**

  ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E6%93%8D%E4%BD%9C%E6%A0%B7%E5%BC%8F.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>操作样式</title>
      <style>
          .cls1{
              background: pink;
              height: 30px;
          }
      </style>
  </head>
  <body>
      <div style="border: 1px solid red;" id="div">我是div</div>
      <input type="button" id="btn1" value="获取div的样式"> &nbsp;&nbsp;
      <input type="button" id="btn2" value="设置div的背景色为蓝色">&nbsp;&nbsp;
      <br><br><br>
      <input type="button" id="btn3" value="给div设置cls1样式"> &nbsp;&nbsp;
      <input type="button" id="btn4" value="给div删除cls1样式"> &nbsp;&nbsp;
      <input type="button" id="btn5" value="给div设置或删除cls1样式"> &nbsp;&nbsp;
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      // 1.css(name)   获取css样式
      $("#btn1").click(function(){
          alert($("#div").css("border"));
      });
  
      // 2.css(name,value)   设置CSS样式
      $("#btn2").click(function(){
          $("#div").css("background","blue");
      });
  
      // 3.addClass(value)   给指定的对象添加样式类名
      $("#btn3").click(function(){
          $("#div").addClass("cls1");
      });
  
      // 4.removeClass(value)  给指定的对象删除样式类名
      $("#btn4").click(function(){
          $("#div").removeClass("cls1");
      });
  
      // 5.toggleClass(value)  如果没有样式类名，则添加。如果有，则删除
      $("#btn5").click(function(){
          $("#div").toggleClass("cls1");
      });
      
  </script>
  </html>
  ```

## 4.4、操作属性

- **常用方法**

  ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E6%93%8D%E4%BD%9C%E5%B1%9E%E6%80%A7.png)

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>操作属性</title>
  </head>
  <body>
      <input type="text" id="username"> 
      <br>
      <input type="button" id="btn1" value="获取输入框的id属性">  &nbsp;&nbsp;
      <input type="button" id="btn2" value="给输入框设置value属性">
      <br><br>
  
      <input type="radio" id="gender1" name="gender">男
      <input type="radio" id="gender2" name="gender">女
      <br>
      <input type="button" id="btn3" value="选中女">
      <br><br>
      
      <select>
          <option>---请选择---</option>
          <option id="bk">本科</option>
          <option id="zk">专科</option>
      </select>
      <br>
      <input type="button" id="btn4" value="选中本科">
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      // 1.attr(name,[value])   获得/设置属性的值
      //按钮一：获取输入框的id属性
      $("#btn1").click(function(){
          alert($("#username").attr("id"));
      });
      
      //按钮二：给输入框设置value属性
      $("#btn2").click(function(){
          $("#username").attr("value","hello...");
      });
      
  
      // 2.prop(name,[value])   获得/设置属性的值(checked，selected)
      //按钮三：选中女
      $("#btn3").click(function(){
          $("#gender2").prop("checked",true);
      });
  
      //按钮四：选中本科
      $("#btn4").click(function(){
          $("#bk").prop("selected",true);
      });
  </script>
  </html>
  ```

## 4.5、小结

- 操作文本
  - html() html(…)：获取或设置标签的文本，解析标签。
- 操作对象
  - $(“元素”)：创建指定元素。
  - append(element)：添加成最后一个子元素，由添加者对象调用。 
  - prepend(element)：添加成第一个子元素，由添加者对象调用。 
  - before(element)：添加到当前元素的前面，两者之间是兄弟关系，由添加者对象调用。
  - after(element)：添加到当前元素的后面，两者之间是兄弟关系，由添加者对象调用。
  - remove()：删除指定元素(自己移除自己)。
- 操作样式
  - addClass(value)：给指定的对象添加样式类名。
  - removeClass(value)：给指定的对象删除样式类名。
- 操作属性
  - attr(name,[value])：获得/设置属性的值。
  - prop(name,[value])：获得/设置属性的值(checked，selected)。

# 5、综合案例 复选框

## 5.1、案例效果

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E5%A4%8D%E9%80%89%E6%A1%86-%E6%A1%88%E4%BE%8B%E6%95%88%E6%9E%9C.png)

## 5.2、分析和实现

**功能分析**

- 全选
  - 1. 为全选按钮绑定单击事件。
    2. 获取所有的商品项复选框元素，为其添加 checked 属性，属性值为 true。
- 全不选
  - 1. 为全不选按钮绑定单击事件。
    2. 获取所有的商品项复选框元素，为其添加 checked 属性，属性值为 false。
- 反选
  - 1. 为反选按钮绑定单击事件
    2. 获取所有的商品项复选框元素，为其添加 checked 属性，属性值是目前相反的状态。

**代码实现**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>复选框</title>
</head>
<body>
    <table id="tab1" border="1" width="800" align="center">
        <tr>
            <th style="text-align: left">
                <input style="background:lightgreen" id="selectAll" type="button" value="全选">
                <input style="background:lightgreen" id="selectNone" type="button" value="全不选">
                <input style="background:lightgreen" id="reverse" type="button" value="反选">
            </th>
    
            <th>分类ID</th>
            <th>分类名称</th>
            <th>分类描述</th>
            <th>操作</th>
        </tr>
        <tr>
            <td><input type="checkbox" class="item"></td>
            <td>1</td>
            <td>手机数码</td>
            <td>手机数码类商品</td>
            <td><a href="">修改</a>|<a href="">删除</a></td>
        </tr>
        <tr>
            <td><input type="checkbox" class="item"></td>
            <td>2</td>
            <td>电脑办公</td>
            <td>电脑办公类商品</td>
            <td><a href="">修改</a>|<a href="">删除</a></td>
        </tr>
        <tr>
            <td><input type="checkbox" class="item"></td>
            <td>3</td>
            <td>鞋靴箱包</td>
            <td>鞋靴箱包类商品</td>
            <td><a href="">修改</a>|<a href="">删除</a></td>
        </tr>
        <tr>
            <td><input type="checkbox" class="item"></td>
            <td>4</td>
            <td>家居饰品</td>
            <td>家居饰品类商品</td>
            <td><a href="">修改</a>|<a href="">删除</a></td>
        </tr>
    </table>
</body>
<script src="js/jquery-3.3.1.min.js"></script>
<script>
    //全选
    //1.为全选按钮添加单击事件
    $("#selectAll").click(function(){
        //2.获取所有的商品复选框元素，为其添加checked属性，属性值true
        $(".item").prop("checked",true);
    });


    //全不选
    //1.为全不选按钮添加单击事件
    $("#selectNone").click(function(){
        //2.获取所有的商品复选框元素，为其添加checked属性，属性值false
        $(".item").prop("checked",false);
    });


    //反选
    //1.为反选按钮添加单击事件
    $("#reverse").click(function(){
        //2.获取所有的商品复选框元素，为其添加checked属性，属性值是目前相反的状态
        let items = $(".item");
        items.each(function(){
            $(this).prop("checked",!$(this).prop("checked"));
        });
    });
</script>
</html>
```

# 6、综合案例 随机图片

## 6.1、案例效果

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E9%9A%8F%E5%8D%B3%E5%9B%BE%E7%89%87-%E5%B0%8F%E5%9B%BE%E7%9A%84%E5%88%87%E6%8D%A2&%E5%A4%A7%E5%9B%BE%E7%9A%84%E5%B1%95%E7%A4%BA.png)

## 6.2、动态切换小图的分析和实现

- **功能分析**

  1. 准备一个数组
  2. 定义计数器
  3. 定义定时器对象
  4. 定义图片路径变量
  5. 为开始按钮绑定单击事件
  6. 设置按钮状态 
  7. 设置定时器，循环显示图片 
  8. 循环获取图片路径 
  9. 将当前图片显示到小图片上 
  10. 计数器自增

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>随机图片</title>
  </head>
  <body>
  <!-- 小图 -->
  <div style="background-color:red;border: dotted; height: 50px; width: 50px">
      <img src="img/01.jpg" id="small" style="width: 50px; height: 50px;">
  </div>
  <!-- 大图 -->
  <div style="border: double ;width: 400px; height: 400px; position: absolute; left: 500px; top:10px">
      <img src="" id="big" style="width: 400px; height: 400px; display:none;">
  </div>
  
  <!-- 开始和结束按钮 -->
  <input id="startBtn" type="button" style="width: 150px;height: 150px; font-size: 20px" value="开始">
  <input id="stopBtn" type="button" style="width: 150px;height: 150px; font-size: 20px" value="停止">
  </body>
  <script src="js/jquery-3.3.1.min.js"></script>
  <script>
      //1.准备一个数组
      let imgs = [
          "img/01.jpg",
          "img/02.jpg",
          "img/03.jpg",
          "img/04.jpg",
          "img/05.jpg",
          "img/06.jpg",
          "img/07.jpg",
          "img/08.jpg",
          "img/09.jpg",
          "img/10.jpg"];
  		
      //2.定义计数器变量
      let count = 0;
      
      //3.声明定时器对象
      let time = null;
      
      //4.声明图片路径变量
      let imgSrc = "";
      
      //5.为开始按钮绑定单击事件
      $("#startBtn").click(function(){
          //6.设置按钮状态
          //禁用开始按钮
          $("#startBtn").prop("disabled",true);
          //启用停止按钮
          $("#stopBtn").prop("disabled",false);
          
          //7.设置定时器，循环显示图片
          time = setInterval(function(){
              //8.循环获取图片路径
              let index = count % imgs.length; // 0%10=0  1%10=1  2%10=2 .. 9%10=9  10%10=0  
                      
              //9.将当前图片显示到小图片上
              imgSrc = imgs[index];
              $("#small").prop("src",imgSrc);
                      
              //10.计数器自增
              count++;
          },10);
      });
  </script>
  </html>
  ```

## 6.3、显示大图的分析和实现

- **功能分析**

  1. 为停止按钮绑定单击事件
  2. 取消定时器
  3. 设置按钮状态 
  4. 将图片显示到大图片上

- **代码实现**

  ```js
  //11.为停止按钮绑定单击事件
  $("#stopBtn").click(function(){
      //12.取消定时器
      clearInterval(time);
  
      //13.设置按钮状态
      //启用开始按钮
      $("#startBtn").prop("disabled",false);
      //禁用停止按钮
      $("#stopBtn").prop("disabled",true);
  
      //14.将图片显示到大图片上
      $("#big").prop("src",imgSrc);
      $("#big").prop("style","width: 400px; height: 400px;");
  });
  ```

# Ajax

**今日目标：**

> * 能够使用 axios 发送 ajax 请求
> * 熟悉 json 格式，并能使用 Fastjson 完成 java 对象和 json 串的相互转换
> * 使用 axios + json 完成综合案例

## 概述

==`AJAX` (Asynchronous JavaScript And XML)：异步的 JavaScript 和 XML。==

我们先来说概念中的 `JavaScript` 和 `XML`，`JavaScript` 表明该技术和前端相关；`XML` 是指以此进行数据交换。而这两个我们之前都学习过。

### 作用

AJAX 作用有以下两方面：

1. **与服务器进行数据交换**：通过AJAX可以给服务器发送请求，服务器将数据直接响应回给浏览器。如下图

我们先来看之前做功能的流程，如下图：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823235114367.png" alt="image-20210823235114367" style="zoom:70%;" />

如上图，`Servlet` 调用完业务逻辑层后将数据存储到域对象中，然后跳转到指定的 `jsp` 页面，在页面上使用 `EL表达式` 和 `JSTL` 标签库进行数据的展示。

而我们学习了AJAX 后，就可以==使用AJAX和服务器进行通信，以达到使用 HTML+AJAX来替换JSP页面==了。如下图，浏览器发送请求servlet，servlet 调用完业务逻辑层后将数据直接响应回给浏览器页面，页面使用 HTML 来进行数据展示。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823235006847.png" alt="image-20210823235006847" style="zoom:70%;" />

2. **异步交互**：可以在==不重新加载整个页面==的情况下，与服务器交换数据并==更新部分网页==的技术，如：搜索联想、用户名是否可用校验，等等…

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824000706401.png" alt="image-20210824000706401" style="zoom:80%;" />

上图所示的效果我们经常见到，在我们输入一些关键字（例如 `奥运`）后就会在下面联想出相关的内容，而联想出来的这部分数据肯定是存储在百度的服务器上，而我们并没有看出页面重新刷新，这就是 ==更新局部页面== 的效果。再如下图：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824001015706.png" alt="image-20210824001015706" style="zoom:80%;" />

我们在用户名的输入框输入用户名，当输入框一失去焦点，如果用户名已经被占用就会在下方展示提示的信息；在这整个过程中也没有页面的刷新，只是在局部展示出了提示信息，这就是 ==更新局部页面== 的效果。

### 同步和异步

知道了局部刷新后，接下来我们再聊聊同步和异步:

* 同步发送请求过程如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824001443897.png" alt="image-20210824001443897" style="zoom:80%;" />

​	浏览器页面在发送请求给服务器，在服务器处理请求的过程中，浏览器页面不能做其他的操作。只能等到服务器响应结束后才能，浏览器页面才能继续做其他的操作。

* 异步发送请求过程如下

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824001608916.png" alt="image-20210824001608916" style="zoom:80%;" />

  浏览器页面发送请求给服务器，在服务器处理请求的过程中，浏览器页面还可以做其他的操作。

## 快速入门

### 服务端实现

在项目的创建 `com.iflytek.web.servlet` ，并在该包下创建名为  `AjaxServlet` 的servlet

```java
@WebServlet("/ajaxServlet")
public class AjaxServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 响应数据
        response.getWriter().write("hello ajax~");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

### 客户端实现

在 `webapp` 下创建名为 `01-ajax-demo1.html` 的页面，在该页面书写 `ajax` 代码

* 创建核心对象，不同的浏览器创建的对象是不同的

  ```js
  var xhttp;
  if (window.XMLHttpRequest) {
      xhttp = new XMLHttpRequest();
  } else {
      // code for IE6, IE5
      xhttp = new ActiveXObject("Microsoft.XMLHTTP");
  }
  ```

* 发送请求

  ```js
  //建立连接
  xhttp.open("GET", "http://localhost:8080/ajax-demo/ajaxServlet");
  //发送请求
  xhttp.send();
  ```

* 获取响应

  ```js
  xhttp.onreadystatechange = function() {
      if (this.readyState == 4 && this.status == 200) {
          // 通过 this.responseText 可以获取到服务端响应的数据
          alert(this.responseText);
      }
  };
  ```

**完整代码如下：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script>
    //1. 创建核心对象
    var xhttp;
    if (window.XMLHttpRequest) {
        xhttp = new XMLHttpRequest();
    } else {
        // code for IE6, IE5
        xhttp = new ActiveXObject("Microsoft.XMLHTTP");
    }
    //2. 发送请求
    xhttp.open("GET", "http://localhost:8080/ajax-demo/ajaxServlet");
    xhttp.send();

    //3. 获取响应
    xhttp.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            alert(this.responseText);
        }
    };
</script>
</body>
</html>
```

### 测试

在浏览器地址栏输入 `http://localhost:8080/ajax-demo/01-ajax-demo1.html` ，在 `01-ajax-demo1.html`加载的时候就会发送 `ajax` 请求，效果如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824005956117.png" alt="image-20210824005956117" style="zoom:67%;" />

我们可以通过 `开发者模式` 查看发送的 AJAX 请求。在浏览器上按 `F12` 快捷键

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824010247642.png" alt="image-20210824010247642" style="zoom:80%;" />

这个是查看所有的请求，如果我们只是想看 异步请求的话，点击上图中 `All` 旁边的 `XHR`，会发现只展示 Type 是 `xhr` 的请求。如下图：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824010438260.png" alt="image-20210824010438260" style="zoom:80%;" /> 

## 案例

需求：在完成用户注册时，当用户名输入框失去焦点时，校验用户名是否在数据库已存在

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210824201415745.png" alt="image-20210824201415745" style="zoom:60%;" />

### 分析

* **前端完成的逻辑**
  1.  给用户名输入框绑定光标失去焦点事件 `onblur`
  2.  发送 ajax请求，携带username参数
  3.  处理响应：是否显示提示信息
* **后端完成的逻辑**
  1. 接收用户名
  2. 调用service查询User。此案例是为了演示前后端异步交互，所以此处我们不做业务逻辑处理
  3. 返回标记

整体流程如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210829183854285.png" alt="image-20210829183854285" style="zoom:80%;" />

### 后端实现

在 `com.iflytek.web.servlet` 包中定义名为 `SelectUserServlet`  的servlet。代码如下：

```java
@WebServlet("/selectUserServlet")
public class SelectUserServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 接收用户名
        String username = request.getParameter("username");
        //2. 调用service查询User对象，此处不进行业务逻辑处理，直接给 flag 赋值为 true，表明用户名占用
        boolean flag = true;
        //3. 响应标记
        response.getWriter().write("" + flag);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

### 前端实现

将 `04-资料\1. 验证用户名案例\1. 静态页面` 下的文件整体拷贝到项目下 `webapp` 下。并在 `register.html` 页面的 `body` 结束标签前编写 `script` 标签，在该标签中实现如下逻辑

**第一步：给用户名输入框绑定光标失去焦点事件 `onblur`**

```js
//1. 给用户名输入框绑定 失去焦点事件
document.getElementById("username").onblur = function () {
    
}
```

**第二步：发送 ajax请求，携带username参数**

在 `第一步` 绑定的匿名函数中书写发送 ajax 请求的代码

```js
//2. 发送ajax请求
//2.1. 创建核心对象
var xhttp;
if (window.XMLHttpRequest) {
    xhttp = new XMLHttpRequest();
} else {
    // code for IE6, IE5
    xhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
//2.2. 发送请求
xhttp.open("GET", "http://localhost:8080/ajax-demo/selectUserServlet);
xhttp.send();

//2.3. 获取响应
xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
        //处理响应的结果
    }
};
```

由于我们发送的是 GET 请求，所以需要在 URL 后拼接从输入框获取的用户名数据。而我们在 `第一步` 绑定的匿名函数中通过以下代码可以获取用户名数据

```js
// 获取用户名的值
var username = this.value;  //this ： 给谁绑定的事件，this就代表谁
```

而携带数据需要将 URL 修改为：

```js
xhttp.open("GET", "http://localhost:8080/ajax-demo/selectUserServlet?username="+username);
```

**第三步：处理响应：是否显示提示信息**

当 `this.readyState == 4 && this.status == 200` 条件满足时，说明已经成功响应数据了。

此时需要判断响应的数据是否是 "true" 字符串，如果是说明用户名已经占用给出错误提示；如果不是说明用户名未被占用清除错误提示。代码如下

```js
//判断
if(this.responseText == "true"){
    //用户名存在，显示提示信息
    document.getElementById("username_err").style.display = '';
}else {
    //用户名不存在 ，清楚提示信息
    document.getElementById("username_err").style.display = 'none';
}
```

**综上所述，前端完成代码如下：**

```js
//1. 给用户名输入框绑定 失去焦点事件
document.getElementById("username").onblur = function () {
    //2. 发送ajax请求
    // 获取用户名的值
    var username = this.value;

    //2.1. 创建核心对象
    var xhttp;
    if (window.XMLHttpRequest) {
        xhttp = new XMLHttpRequest();
    } else {
        // code for IE6, IE5
        xhttp = new ActiveXObject("Microsoft.XMLHTTP");
    }
    //2.2. 发送请求
    xhttp.open("GET", "http://localhost:8080/ajax-demo/selectUserServlet?username="+username);
    xhttp.send();

    //2.3. 获取响应
    xhttp.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            //alert(this.responseText);
            //判断
            if(this.responseText == "true"){
                //用户名存在，显示提示信息
                document.getElementById("username_err").style.display = '';
            }else {
                //用户名不存在 ，清楚提示信息
                document.getElementById("username_err").style.display = 'none';
            }
        }
    };
}
```



# JQuery的Ajax

## JQuery的GET方式实现AJAX

- **核心语法：**$.get(url,[data],[callback],[type]); 

  - url：请求的资源路径。
  - data：发送给服务器端的请求参数，格式可以是key=value，也可以是 js 对象。
  - callback：当请求成功后的回调函数，可以在函数中编写我们的逻辑代码。 
  - type：预期的返回数据的类型，取值可以是 xml, html, js, json, text等。

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>用户注册</title>
  </head>
  <body>
      <form autocomplete="off">
          姓名：<input type="text" id="username">
          <span id="uSpan"></span>
          <br>
          密码：<input type="password" id="password">
          <br>
          <input type="submit" value="注册">
      </form>
  </body>
  <script src="js/jquery-3.1.1.min.js"></script>
  <script>
      //1.为用户名绑定失去焦点事件
      $("#username").blur(function () {
          let username = $("#username").val();
          //2.jQuery的GET方式实现AJAX
          $.get(
              //请求的资源路径
              "userServlet",
              //请求参数
              "username=" + username,
              //回调函数
              function (data) {
                  //将响应的数据显示到span标签
                  $("#uSpan").html(data);
              },
              //响应数据形式
              "text"
          );
      });
  </script>
  </html>
  ```

## JQuery的POST方式实现AJAX

- **核心语法：**$.post(url,[data],[callback],[type]); 

  - url：请求的资源路径。
  - data：发送给服务器端的请求参数，格式可以是key=value，也可以是 js 对象。 
  - callback：当请求成功后的回调函数，可以在函数中编写我们的逻辑代码。
  - type：预期的返回数据的类型，取值可以是 xml, html, js, json, text等。

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>用户注册</title>
  </head>
  <body>
      <form autocomplete="off">
          姓名：<input type="text" id="username">
          <span id="uSpan"></span>
          <br>
          密码：<input type="password" id="password">
          <br>
          <input type="submit" value="注册">
      </form>
  </body>
  <script src="js/jquery-3.1.1.min.js"></script>
  <script>
      //1.为用户名绑定失去焦点事件
      $("#username").blur(function () {
          let username = $("#username").val();
          //2.jQuery的POST方式实现AJAX
          $.post(
              //请求的资源路径
              "userServlet",
              //请求参数
              "username=" + username,
              //回调函数
              function (data) {
                  //将响应的数据显示到span标签
                  $("#uSpan").html(data);
              },
              //响应数据形式
              "text"
          );
      });
  </script>
  </html>
  ```

## JQuery的通用方式实现AJAX

- **核心语法：**$.ajax({name:value,name:value,…}); 

  - url：请求的资源路径。
  - async：是否异步请求，true-是，false-否 (默认是 true)。
  - data：发送到服务器的数据，可以是键值对形式，也可以是 js 对象形式。
  - type：请求方式，POST 或 GET (默认是 GET)。
  - dataType：预期的返回数据的类型，取值可以是 xml, html, js, json, text等。 
  - success：请求成功时调用的回调函数。
  - error：请求失败时调用的回调函数。

- **代码实现**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>用户注册</title>
  </head>
  <body>
      <form autocomplete="off">
          姓名：<input type="text" id="username">
          <span id="uSpan"></span>
          <br>
          密码：<input type="password" id="password">
          <br>
          <input type="submit" value="注册">
      </form>
  </body>
  <script src="js/jquery-3.1.1.min.js"></script>
  <script>
      //1.为用户名绑定失去焦点事件
      $("#username").blur(function () {
          let username = $("#username").val();
          //2.jQuery的通用方式实现AJAX
          $.ajax({
              //请求资源路径
              url:"userServletxxx",
              //是否异步
              async:true,
              //请求参数
              data:"username="+username,
              //请求方式
              type:"POST",
              //数据形式
              dataType:"text",
              //请求成功后调用的回调函数
              success:function (data) {
                  //将响应的数据显示到span标签
                  $("#uSpan").html(data);
              },
              //请求失败后调用的回调函数
              error:function () {
                  alert("操作失败...");
              }
          });
      });
  </script>
  </html>
  ```

# axios

Axios 对原生的AJAX进行封装，简化书写。

Axios官网是：`https://www.axios-http.cn`

## 基本使用

axios 使用是比较简单的，分为以下两步：

* 引入 axios 的 js 文件

  ```html
  <script src="js/axios-0.18.0.js"></script>
  ```

* 使用axios 发送请求，并获取响应结果

  * 发送 get 请求

    ```js
    axios({
        method:"get",
        url:"http://localhost:8080/ajax-demo1/aJAXDemo1?username=zhangsan"
    }).then(function (resp){
        alert(resp.data);
    })
    ```

  * 发送 post 请求

    ```js
    axios({
        method:"post",
        url:"http://localhost:8080/ajax-demo1/aJAXDemo1",
        data:"username=zhangsan"
    }).then(function (resp){
        alert(resp.data);
    });
    ```

`axios()` 是用来发送异步请求的，小括号中使用 js 对象传递请求相关的参数：

* `method` 属性：用来设置请求方式的。取值为 `get` 或者 `post`。
* `url` 属性：用来书写请求的资源路径。如果是 `get` 请求，需要将请求参数拼接到路径的后面，格式为： `url?参数名=参数值&参数名2=参数值2`。
* `data` 属性：作为请求体被发送的数据。也就是说如果是 `post` 请求的话，数据需要作为 `data` 属性的值。

`then()` 需要传递一个匿名函数。我们将 `then()` 中传递的匿名函数称为 ==回调函数==，意思是该匿名函数在发送请求时不会被调用，而是在成功响应后调用的函数。而该回调函数中的 `resp` 参数是对响应的数据进行封装的对象，通过 `resp.data` 可以获取到响应的数据。

## 快速入门

### 后端实现

定义一个用于接收请求的servlet，代码如下：

```java
@WebServlet("/axiosServlet")
public class AxiosServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("get...");
        //1. 接收请求参数
        String username = request.getParameter("username");
        System.out.println(username);
        //2. 响应数据
        response.getWriter().write("hello Axios~");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("post...");
        this.doGet(request, response);
    }
}
```

### 前端实现

* 引入 js 文件

  ```js
  <script src="js/axios-0.18.0.js"></script>
  ```

* 发送 ajax 请求

  * get 请求

    ```js
    axios({
        method:"get",
        url:"http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan"
    }).then(function (resp) {
        alert(resp.data);
    })
    ```

  * post 请求

    ```js
    axios({
        method:"post",
        url:"http://localhost:8080/ajax-demo/axiosServlet",
        data:"username=zhangsan"
    }).then(function (resp) {
        alert(resp.data);
    })
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

<script src="js/axios-0.18.0.js"></script>
<script>
    //1. get
   /* axios({
        method:"get",
        url:"http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan"
    }).then(function (resp) {
        alert(resp.data);
    })*/

    //2. post  在js中{} 表示一个js对象，而这个js对象中有三个属性
    axios({
        method:"post",
        url:"http://localhost:8080/ajax-demo/axiosServlet",
        data:"username=zhangsan"
    }).then(function (resp) {
        alert(resp.data);
    })
</script>
</body>
</html>
```

## 请求方法别名

为了方便起见， Axios 已经为所有支持的请求方法提供了别名。如下：

* `get` 请求 ： `axios.get(url[,config])`

* `delete` 请求 ： `axios.delete(url[,config])`

* `head` 请求 ： `axios.head(url[,config])`

* `options` 请求 ： `axios.option(url[,config])`

* `post` 请求：`axios.post(url[,data[,config])`

* `put` 请求：`axios.put(url[,data[,config])`

* `patch` 请求：`axios.patch(url[,data[,config])`

而我们只关注 `get` 请求和 `post` 请求。

入门案例中的 `get` 请求代码可以改为如下：

```js
axios.get("http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan").then(function (resp) {
    alert(resp.data);
});
```

入门案例中的 `post` 请求代码可以改为如下：

```js
axios.post("http://localhost:8080/ajax-demo/axiosServlet","username=zhangsan").then(function (resp) {
    alert(resp.data);
})
```



# JSON

## 概述

==概念：`JavaScript Object Notation`。JavaScript 对象表示法.==

如下是 `JavaScript` 对象的定义格式：

```js
{
	name:"zhangsan",
	age:23,
	city:"北京"
}
```

接下来我们再看看 `JSON` 的格式：

```json
{
	"name":"zhangsan",
	"age":23,
	"city":"北京"
}
```

通过上面 js 对象格式和 json 格式进行对比，发现两个格式特别像。只不过 js 对象中的属性名可以使用引号（可以是单引号，也可以是双引号）；而 `json` 格式中的键要求必须使用双引号括起来，这是 `json` 格式的规定。`json` 格式的数据有什么作用呢？

作用：由于其语法格式简单，层次结构鲜明，现多用于作为==数据载体==，在网络中进行数据传输。如下图所示就是服务端给浏览器响应的数据，这个数据比较简单，如果现需要将 JAVA 对象中封装的数据响应回给浏览器的话，应该以何种数据传输呢？

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210830232718632.png" alt="image-20210830232718632" style="zoom:80%;" />

大家还记得 `ajax` 的概念吗？ 是 ==异步的 JavaScript 和 xml==。这里的 xml就是以前进行数据传递的方式，如下：

```xml
<student>
    <name>张三</name>
    <age>23</age>
    <city>北京</city>
</student>
```

再看 `json` 描述以上数据的写法：

```json
{	
	"name":"张三",
    "age":23,
    "city":"北京"
}
```

上面两种格式进行对比后就会发现 `json` 格式数据的简单，以及所占的字节数少等优点。

## JSON 基础语法

### 定义格式

`JSON` 本质就是一个字符串，但是该字符串内容是有一定的格式要求的。 定义格式如下：

```js
var 变量名 = '{"key":value,"key":value,...}';
```

`JSON` 串的键要求必须使用双引号括起来，而值根据要表示的类型确定。value 的数据类型分为如下

* 数字（整数或浮点数）
* 字符串（使用双引号括起来）
* 逻辑值（true或者false）
* 数组（在方括号中）
* 对象（在花括号中）
* null

示例：

```js
var jsonStr = '{"name":"zhangsan","age":23,"addr":["北京","上海","西安"]}'
```

### 代码演示

创建一个页面，在该页面的 `<script>` 标签中定义json字符串

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    //1. 定义JSON字符串
    var jsonStr = '{"name":"zhangsan","age":23,"addr":["北京","上海","西安"]}'
    alert(jsonStr);

</script>
</body>
</html>
```

通过浏览器打开，页面效果如下图所示

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831223339530.png" alt="image-20210831223339530" style="zoom:80%;" />

现在我们需要获取到该 `JSON` 串中的 `name` 属性值，应该怎么处理呢？

如果它是一个 js 对象，我们就可以通过 `js对象.属性名` 的方式来获取数据。JS 提供了一个对象 `JSON` ，该对象有如下两个方法：

* `parse(str)` ：将 JSON串转换为 js 对象。使用方式是： ==`var jsObject = JSON.parse(jsonStr);`==
* `stringify(obj)` ：将 js 对象转换为 JSON 串。使用方式是：==`var jsonStr = JSON.stringify(jsObject)`==

代码演示：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    //1. 定义JSON字符串
    var jsonStr = '{"name":"zhangsan","age":23,"addr":["北京","上海","西安"]}'
    alert(jsonStr);

    //2. 将 JSON 字符串转为 JS 对象
    let jsObject = JSON.parse(jsonStr);
    alert(jsObject)
    alert(jsObject.name)
    //3. 将 JS 对象转换为 JSON 字符串
    let jsonStr2 = JSON.stringify(jsObject);
    alert(jsonStr2)
</script>
</body>
</html>
```

### 发送异步请求携带参数

后面我们使用 `axios` 发送请求时，如果要携带复杂的数据时都会以 `JSON` 格式进行传递，如下

```js
axios({
    method:"post",
    url:"http://localhost:8080/ajax-demo/axiosServlet",
    data:"username=zhangsan"
}).then(function (resp) {
    alert(resp.data);
})
```

请求参数不可能由我们自己拼接字符串吧？肯定不用，可以提前定义一个 js 对象，用来封装需要提交的参数，然后使用 `JSON.stringify(js对象)` 转换为 `JSON` 串，再将该 `JSON` 串作为 `axios` 的 `data` 属性值进行请求参数的提交。如下：

```js
var jsObject = {name:"张三"};

axios({
    method:"post",
    url:"http://localhost:8080/ajax-demo/axiosServlet",
    data: JSON.stringify(jsObject)
}).then(function (resp) {
    alert(resp.data);
})
```

而 `axios` 是一个很强大的工具。我们只需要将需要提交的参数封装成 js 对象，并将该 js 对象作为 `axios` 的 `data` 属性值进行，它会自动将 js 对象转换为 `JSON` 串进行提交。如下：

```js
var jsObject = {name:"张三"};

axios({
    method:"post",
    url:"http://localhost:8080/ajax-demo/axiosServlet",
    data:jsObject  //这里 axios 会将该js对象转换为 json 串的
}).then(function (resp) {
    alert(resp.data);
})
```

> ==注意：==
>
> * js 提供的 `JSON` 对象我们只需要了解一下即可。因为 `axios` 会自动对 js 对象和 `JSON` 串进行想换转换。
> * 发送异步请求时，如果请求参数是 `JSON` 格式，那请求方式必须是 `POST`。因为 `JSON` 串需要放在请求体中。

## JSON串和Java对象的相互转换

学习完 json 后，接下来聊聊 json 的作用。以后我们会以 json 格式的数据进行前后端交互。前端发送请求时，如果是复杂的数据就会以 json 提交给后端；而后端如果需要响应一些复杂的数据时，也需要以 json 格式将数据响应回给浏览器。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831104901912.png" alt="image-20210831104901912" style="zoom:70%;" />

在后端我们就需要重点学习以下两部分操作：

* 请求数据：JSON字符串转为Java对象
* 响应数据：Java对象转为JSON字符串

接下来给大家介绍一套 API，可以实现上面两部分操作。这套 API 就是 `Fastjson`

### Fastjson 概述

`Fastjson` 是阿里巴巴提供的一个Java语言编写的高性能功能完善的 `JSON` 库，是目前Java语言中最快的 `JSON` 库，可以实现 `Java` 对象和 `JSON` 字符串的相互转换。

###  Fastjson 使用

`Fastjson` 使用也是比较简单的，分为以下三步完成

1. **导入坐标**

   ```xml
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>fastjson</artifactId>
       <version>1.2.62</version>
   </dependency>
   ```

2. **Java对象转JSON**

   ```java
   String jsonStr = JSON.toJSONString(obj);
   ```

   将 Java 对象转换为 JSON 串，只需要使用 `Fastjson` 提供的 `JSON` 类中的 `toJSONString()` 静态方法即可。

3. **JSON字符串转Java对象**

   ```java
   User user = JSON.parseObject(jsonStr, User.class);
   ```

   将 json 转换为 Java 对象，只需要使用 `Fastjson` 提供的 `JSON` 类中的 `parseObject()` 静态方法即可。

### 代码演示

* 引入坐标

* 创建一个类，专门用来测试 Java 对象和 JSON 串的相互转换，代码如下：

  ```java
  public class FastJsonDemo {
  
      public static void main(String[] args) {
          //1. 将Java对象转为JSON字符串
          User user = new User();
          user.setId(1);
          user.setUsername("zhangsan");
          user.setPassword("123");
  
          String jsonString = JSON.toJSONString(user);
          System.out.println(jsonString);//{"id":1,"password":"123","username":"zhangsan"}
  
  
          //2. 将JSON字符串转为Java对象
          User u = JSON.parseObject("{\"id\":1,\"password\":\"123\",\"username\":\"zhangsan\"}", User.class);
          System.out.println(u);
      }
  }
  ```

  

# 案例

## 需求

使用Axios + JSON 完成品牌列表数据查询和添加。页面效果还是下图所示：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210830234803335.png" alt="image-20210830234803335" style="zoom:60%;" />

## 查询所有功能

![image-20210831085332612](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831085332612.png)

如上图所示就该功能的整体流程。前后端需以 JSON 格式进行数据的传递；由于此功能是查询所有的功能，前端发送 ajax 请求不需要携带参数，而后端响应数据需以如下格式的 json 数据

![image-20210831090839336](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831090839336.png)

### 环境准备

将 `02-AJAX\04-资料\3. 品牌列表案例\初始工程` 下的 `brand-demo` 工程拷贝到我们自己 `工作空间` ，然后再将项目导入到我们自己的 Idea 中。工程目录结构如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230409100650325.png" alt="image-20230409100650325" style="zoom: 67%;" />

==注意：==

* 在给定的原始工程中已经给定一些代码。而在此案例中我们只关注前后端交互代码实现
* 要根据自己的数据库环境去修改连接数据库的信息，在 `mybatis-config.xml` 核心配置文件中修改

### 后端实现

在 `com.iflytek.web` 包下创建名为 `SelectAllServlet` 的 `servlet`，具体的逻辑如下：

* 调用 service 的 `selectAll()` 方法进行查询所有的逻辑处理
* 将查询到的集合数据转换为 json 数据。我们将此过程称为 ==序列化==；如果是将 json 数据转换为 Java 对象，我们称之为 ==反序列化==
* 将 json 数据响应回给浏览器。这里一定要设置响应数据的类型及字符集 `response.setContentType("text/json;charset=utf-8");`

`SelectAllServlet` 代码如下：

```java
@WebServlet("/selectAllServlet")
public class SelectAllServlet extends HttpServlet {
    private BrandService brandService = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 调用Service查询
        List<Brand> brands = brandService.selectAll();

        //2. 将集合转换为JSON数据   序列化
        String jsonString = JSON.toJSONString(brands);

        //3. 响应数据  application/json   text/json
        response.setContentType("text/json;charset=utf-8");
        response.getWriter().write(jsonString);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```



### 前端实现

1. **引入 js 文件**

在 `brand.html` 页面引入 `axios` 的 js 文件

```html
<script src="js/axios-0.18.0.js"></script>
```

2. **绑定 `页面加载完毕` 事件**

在 `brand.html` 页面绑定加载完毕事件，该事件是在页面加载完毕后被触发，代码如下

```js
window.onload = function() {
	
}
```

3. **发送异步请求**

在页面加载完毕事件绑定的匿名函数中发送异步请求，代码如下：

```js
 //2. 发送ajax请求
axios({
    method:"get",
    url:"http://localhost:8080/brand-demo/selectAllServlet"
}).then(function (resp) {

});
```

4. **处理响应数据**

在 `then` 中的回调函数中通过 `resp.data` 可以获取响应回来的数据，而数据格式如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831093617083.png" alt="image-20210831093617083" style="zoom:80%;" />

现在我们需要拼接字符串，将下面表格中的所有的 `tr` 拼接到一个字符串中，然后使用 `document.getElementById("brandTable").innerHTML = 拼接好的字符串`  就可以动态的展示出用户想看到的数据

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831093938057.png" alt="image-20210831093938057" style="zoom:70%;" />

而表头行是固定的，所以先定义初始值是表头行数据的字符串，如下

```js
//获取数据
let brands = resp.data;
let tableData = " <tr>\n" +
    "        <th>序号</th>\n" +
    "        <th>品牌名称</th>\n" +
    "        <th>企业名称</th>\n" +
    "        <th>排序</th>\n" +
    "        <th>品牌介绍</th>\n" +
    "        <th>状态</th>\n" +
    "        <th>操作</th>\n" +
    "    </tr>";
```

接下来遍历响应回来的数据 `brands` ，拿到每一条品牌数据

```js
for (let i = 0; i < brands.length ; i++) {
    let brand = brands[i];
	
}
```

紧接着就是从 `brand` 对象中获取数据并且拼接 `数据行`，累加到 `tableData` 字符串变量中

```js
tableData += "\n" +
    "    <tr align=\"center\">\n" +
    "        <td>"+(i+1)+"</td>\n" +
    "        <td>"+brand.brandName+"</td>\n" +
    "        <td>"+brand.companyName+"</td>\n" +
    "        <td>"+brand.ordered+"</td>\n" +
    "        <td>"+brand.description+"</td>\n" +
    "        <td>"+brand.status+"</td>\n" +
    "\n" +
    "        <td><a href=\"#\">修改</a> <a href=\"#\">删除</a></td>\n" +
    "    </tr>";
```

最后再将拼接好的字符串写到表格中

```js
// 设置表格数据
document.getElementById("brandTable").innerHTML = tableData;
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
<a href="addBrand.html"><input type="button" value="新增"></a><br>
<hr>
<table id="brandTable" border="1" cellspacing="0" width="100%">
   
</table>

<script src="js/axios-0.18.0.js"></script>

<script>
    //1. 当页面加载完成后，发送ajax请求
    window.onload = function () {
        //2. 发送ajax请求
        axios({
            method:"get",
            url:"http://localhost:8080/brand-demo/selectAllServlet"
        }).then(function (resp) {
            //获取数据
            let brands = resp.data;
            let tableData = " <tr>\n" +
                "        <th>序号</th>\n" +
                "        <th>品牌名称</th>\n" +
                "        <th>企业名称</th>\n" +
                "        <th>排序</th>\n" +
                "        <th>品牌介绍</th>\n" +
                "        <th>状态</th>\n" +
                "        <th>操作</th>\n" +
                "    </tr>";

            for (let i = 0; i < brands.length ; i++) {
                let brand = brands[i];

                tableData += "\n" +
                    "    <tr align=\"center\">\n" +
                    "        <td>"+(i+1)+"</td>\n" +
                    "        <td>"+brand.brandName+"</td>\n" +
                    "        <td>"+brand.companyName+"</td>\n" +
                    "        <td>"+brand.ordered+"</td>\n" +
                    "        <td>"+brand.description+"</td>\n" +
                    "        <td>"+brand.status+"</td>\n" +
                    "\n" +
                    "        <td><a href=\"#\">修改</a> <a href=\"#\">删除</a></td>\n" +
                    "    </tr>";
            }
            // 设置表格数据
            document.getElementById("brandTable").innerHTML = tableData;
        })
    }
</script>
</body>
</html>
```

## 添加品牌功能

![image-20210831100117014](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831100117014.png)

如上所示，当我们点击 `新增` 按钮，会跳转到 `addBrand.html` 页面。在 `addBrand.html` 页面输入数据后点击 `提交` 按钮，就会将数据提交到后端，而后端将数据保存到数据库中。

具体的前后端交互的流程如下：

![image-20210831100329698](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831100329698.png)

==说明：==

前端需要将用户输入的数据提交到后端，这部分数据需要以 json 格式进行提交，数据格式如下：

![image-20210831101234467](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831101234467.png)

### 后端实现

在 `com.iflytek.web` 包下创建名为 `AddServlet` 的 `servlet`，具体的逻辑如下：

* 获取请求参数

  由于前端提交的是 json 格式的数据，所以我们不能使用 `request.getParameter()` 方法获取请求参数

  * 如果提交的数据格式是 `username=zhangsan&age=23` ，后端就可以使用 `request.getParameter()` 方法获取
  * 如果提交的数据格式是 json，后端就需要通过 request 对象获取输入流，再通过输入流读取数据 

* 将获取到的请求参数（json格式的数据）转换为 `Brand` 对象

* 调用 service 的 `add()` 方法进行添加数据的逻辑处理

* 将 json 数据响应回给浏览器。

`AddServlet` 代码如下：

```java
@WebServlet("/addServlet")
public class AddServlet extends HttpServlet {

    private BrandService brandService = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        //1. 接收数据,request.getParameter 不能接收json的数据
       /* String brandName = request.getParameter("brandName");
        System.out.println(brandName);*/

        // 获取请求体数据
        BufferedReader br = request.getReader();
        String params = br.readLine();
        // 将JSON字符串转为Java对象
        Brand brand = JSON.parseObject(params, Brand.class);
        //2. 调用service 添加
        brandService.add(brand);
        //3. 响应成功标识
        response.getWriter().write("success");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

### 前端实现

在 `addBrand.html` 页面给 `提交` 按钮绑定点击事件，并在绑定的匿名函数中发送异步请求，代码如下：

```js
//1. 给按钮绑定单击事件
document.getElementById("btn").onclick = function () {
    //2. 发送ajax请求
    axios({
        method:"post",
        url:"http://localhost:8080/brand-demo/addServlet",
        data:???
    }).then(function (resp) {
        // 判断响应数据是否为 success
        if(resp.data == "success"){
            location.href = "http://localhost:8080/brand-demo/brand.html";
        }
    })
}
```

现在我们只需要考虑如何获取页面上用户输入的数据即可。

首先我们先定义如下的一个 js 对象，该对象是用来封装页面上输入的数据，并将该对象作为上面发送异步请求时 `data` 属性的值。

```js
// 将表单数据转为json
var formData = {
    brandName:"",
    companyName:"",
    ordered:"",
    description:"",
    status:"",
};
```

接下来获取输入框输入的数据，并将获取到的数据赋值给 `formData` 对象指定的属性。比如获取用户名的输入框数据，并把该数据赋值给 `formData` 对象的 `brandName` 属性

```js
// 获取表单数据
let brandName = document.getElementById("brandName").value;
// 设置数据
formData.brandName = brandName;
```

==说明：其他的输入框都用同样的方式获取并赋值。==但是有一个比较特殊，就是状态数据，如下图是页面内容

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210831103843798.png" alt="image-20210831103843798" style="zoom:80%;" />

我们需要判断哪儿个被选中，再将选中的单选框数据赋值给 `formData` 对象的 `status` 属性，代码实现如下：

```js
let status = document.getElementsByName("status");
for (let i = 0; i < status.length; i++) {
    if(status[i].checked){
        //
        formData.status = status[i].value ;
    }
}
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
<h3>添加品牌</h3>
<form action="" method="post">
    品牌名称：<input id="brandName" name="brandName"><br>
    企业名称：<input id="companyName" name="companyName"><br>
    排序：<input id="ordered" name="ordered"><br>
    描述信息：<textarea rows="5" cols="20" id="description" name="description"></textarea><br>
    状态：
    <input type="radio" name="status" value="0">禁用
    <input type="radio" name="status" value="1">启用<br>

    <input type="button" id="btn"  value="提交">
</form>

<script src="js/axios-0.18.0.js"></script>

<script>
    //1. 给按钮绑定单击事件
    document.getElementById("btn").onclick = function () {
        // 将表单数据转为json
        var formData = {
            brandName:"",
            companyName:"",
            ordered:"",
            description:"",
            status:"",
        };
        // 获取表单数据
        let brandName = document.getElementById("brandName").value;
        // 设置数据
        formData.brandName = brandName;

        // 获取表单数据
        let companyName = document.getElementById("companyName").value;
        // 设置数据
        formData.companyName = companyName;

        // 获取表单数据
        let ordered = document.getElementById("ordered").value;
        // 设置数据
        formData.ordered = ordered;

        // 获取表单数据
        let description = document.getElementById("description").value;
        // 设置数据
        formData.description = description;

        let status = document.getElementsByName("status");
        for (let i = 0; i < status.length; i++) {
            if(status[i].checked){
                //
                formData.status = status[i].value ;
            }
        }
        //console.log(formData);
        //2. 发送ajax请求
        axios({
            method:"post",
            url:"http://localhost:8080/brand-demo/addServlet",
            data:formData
        }).then(function (resp) {
            // 判断响应数据是否为 success
            if(resp.data == "success"){
                location.href = "http://localhost:8080/brand-demo/brand.html";
            }
        })
    }
</script>
</body>
</html>
```

==说明：==

`查询所有` 功能和 `添加品牌` 功能就全部实现，大家肯定会感觉前端的代码很复杂；而这只是暂时的，后面学习了 `vue` 前端框架后，这部分前端代码就可以进行很大程度的简化。







