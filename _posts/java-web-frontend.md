---
name: web-front-end
title: Web 前端
date: 2024-09-09
tags: 
- regex
- vue
- js/html/css
categories: web development
---



# HTML

**H**yper**T**ext **M**arkup **L**anguage 

结构：HTML

[HTML 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/html/html-tutorial.html) 

[HTML - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/HTML) 

[HTML（超文本标记语言） | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTML) 

![image-20240926154402465](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926154402465.png)

## 标签

### 基础标签

![image-20240926165557803](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926165557803.png)

![image-20240926170603004](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926170603004.png)

### 图片，音视频标签

![image-20240926170628097](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926170628097.png)

height weight 有 px 和百分比两种写法 src路径

URL 统一资源定位符 ../ 代表上一级目录 

### 超链接标签

![image-20240926171504599](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926171504599.png)

a href="url" 

### 列表标签

![image-20240926171804215](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926171804215.png)

### 表格标签

![image-20240926171833199](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926171833199.png)

边框宽度+空白区间宽度

![image-20240926172249995](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926172249995.png)

### 布局标签

<div></div>


div块级标签，占一整行

span 只占据自己的空间

### 表单标签

![image-20240926193335810](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926193335810.png)

提交表单：方式get/post 

![image-20240926193711640](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926193711640.png)

![image-20240926194217797](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926194217797.png)

**点击框外文本就能让框处于被选中的状态**：

如果要随着表单一起提交，就要起一个name

用**label**标签把用户名包裹起来，给输入框起一个唯一的id, label for “id” 

**radio**：单选按钮，让其name相同达到互斥效果，给他赋值value让服务器接收到有效信息

hidden 修改商品信息，id隐藏在表单里面,提交表单会上传

![image-20240928193146331](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928193146331.png)

submit reset button 如果要指定按钮名字 value

<select>
    <option>123</option>
     <option>4567</option>
</select>


textarea cols支持的列数 rows支持的行数



# CSS

表现：CSS

[CSS 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/css/css-tutorial.html) 

[CSS - 维基百科，自由的百科全书 (wikipedia.org) ](https://zh.wikipedia.org/wiki/CSS) 

[CSS：层叠样式表 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/CSS)

## 导入HTML

在head标签中导入

内联，内部，外部

![image-20240926201958367](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926201958367.png)

`<link href="css路径" rel="stylesheet">`

## CSS选择器

![image-20240926202419738](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926202419738.png)

name唯一 class不唯一

# JavaScript

行为：javascript

什么是JS？

[JavaScript 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/js/js-tutorial.html)

[JavaScript - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/JavaScript)

[JavaScript——动态客户端脚本语言 - 学习 Web 开发 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript) 

![image-20240926202813408](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926202813408.png)

改变HTML 交互性

引入方式：

![image-20240929131050714](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929131050714.png)

内部脚本,一般放在body下面

外部脚本, 不支持自闭合

## 基础语法

- 区分大小写，分号可有可无
- 单行，多行注释
- 大括号表示代码块



### 输出语句

- (window.)alert()弹出警告
- document.write() 写入HTML页面
- console.log() 写入浏览器控制台



### 变量

var是变量，弱类型语言，可以存放不同类型的值

作用域：全局变量而且可以重复声明定义

![image-20240929132314244](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929132314244.png)

let 作用域只在代码块之内 局部变量 不可重复定义

const 常量

### 数据类型

![image-20240929132717037](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929132717037.png)

number boolean string 空（null）默认初始值（undefined）

### 运算符

![image-20240929132954368](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929132954368.png)

全等于(===) 类型不一样返回false，更加严格

等于（==）判断类型是否一样，如果不一样则转换类型，然后才比较值

### 类型转换

转为number：

- string
  - `var str = "20"   parseInt(str)`
  - 字面值转为数字，如果字面值不是数字，转为NaN
- boolean 
  - true->1 false->0

转成boolean

- number： 0和NaN转为false
- string：空字符串转为false
- null&undefined：转为false

### 流程控制

if  switch  while  do while  

![image-20240929135023368](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929135023368.png)

### 函数

- 关键字：function
- 返回值类型：不需要，直接return
- 参数列表：不用写参数类型

![image-20240929135258270](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929135258270.png)

实际调用可以有很多实参，

![image-20240929135611640](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929135611640.png)

add(1,2) = 3

add(1)  = NaN

## JavaScript 对象

### Array

```javascript
//定义数组
var arr1 = new Array(1,2,3);
var arr2 = [1,2,3];
//访问数组
arr1[1]=2;
```

与java不同，js的数组是集合，变长，变类型

```js
//变长
var arr3 = [1,2,3];
arr3[5] = 10;
//索引为5的元素被赋值为10, 中间空出来的是undefined
//变类型
arr3[10] = "Hello";
```

长度：length

增：push(10) 加入值为10的元素

删：splice(0,3)从索引0开始，删除3个元素

### String

length:长度

trim(): 去除字符串两端的空白字符

charAt(i):返回指定位置的字符

indexOf(c) 返回字符第一次出现的索引

### 自定义

```js
var person={
	name:"ZS",
    age:23,
    eat:function(a,b){
        alert("eat"+a+b);
    }
}
person.eat(1,2);
```

### BOM

浏览器对象模型

Navigator Screen

#### window

- 窗口对象，使用window获取
- `alert()` 警告
- `confirm("对话框内容")` 确认/取消对话框
  - 有返回值，返回的是一个boolean
- `setInterval(function,ms)` 定时器，循环执行function
- `setTimeout(function,ms)` 倒计时，只有一次

根据变化的数字，产生固定个数的值，取模

##### history

- 通过window调取，window可省略

- `history.back()`加载前一个URL
- `history.forward()`加载下一个URL

##### location	

- 通过window调取，可省略
- `location.href` 设置当前的URL，可以实现跳转

![image-20240929143103867](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929143103867.png)

### DOM

文档对象模型 W3C指定的XML HTML的规则

查文档

#### document

##### 获取Element

![image-20240929143507450](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929143507450.png)

- id: 每个元素唯一的id
- tagName: 标签名
- class: 同类的，一个元素可以有多个类，一个类可以有多个元素
- name: 用于radio checkbox 和 form表单 提交将输入的值发送

```html
<form action="submit.php" method="post">
<input type="text" name="username" placeholder="Enter your username">
<input type="submit" value="Submit">
</form>
<input type="checkbox" name="fruits" value="apple"> Apple
<input type="checkbox" name="fruits" value="banana"> Banana
<input type="checkbox" name="fruits" value="orange"> Orange
```

##### element通用方法

- element.**style** 获取style标签
  - element.style.color = "red" 设置元素的style中 color属性为red
  - 

- element.**innerHTML** 改变两个标签内部的文本内容



checkbox.**checked**=true 复选框设置为勾选  

## 事件监听 EventListener

事件：发生在HTML元素上的事件，比如按钮被电击，鼠标移到元素之上，按下键盘按键

监听：侦测到事件，执行某些代码

### 事件绑定

#### HTML标签的属性绑定

耦合度太高

#### DOM元素属性绑定

![image-20240929145651893](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929145651893.png)

属性和函数绑定



### 常见的事件

`onblur`: 失去焦点

`onfocus`: 获得焦点

`onchange`:文本改变

`onkeydown`: 按键

`onmouseover`: 鼠标移动到指定对象上

`onmouseout`：鼠标离开指定对象

`onsubmit`: 表单验证，绑定的函数要有返回值，如果返回true则表单会被提交

核心宗旨：把一些交互逻辑更多地放到浏览器这边，减少服务端的压力

![image-20240929152143546](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929152143546.png)

![image-20240929153455050](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929153455050.png)

绑定函数和事件



# Regex 

[正则表达式 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/正则表达式)

[正则表达式 – 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/regexp/regexp-tutorial.html)

[regex101: build, test, and debug regex](https://regex101.com/) 

**正则表达式（Regular Expression, regex）**是一种用来描述和匹配字符串模式的工具，其原理基于形式语言和自动机理论。以下是正则表达式的主要原理和工作机制：

## 基本原理

1. **字符匹配**：正则表达式中的普通字符（如 `a`, `b`, `1`, `2`）直接匹配自身。例如，正则表达式 `abc` 将匹配字符串 "abc"。
2. **元字符和转义**：元字符（如 `.`、`*`、`+`、`?`、`|`、`()`、`[]`、`{}`）有特殊含义。若要匹配这些字符本身，需要使用转义字符 `\`。例如，`\.` 匹配字符 `.`。
3. **字符类**：使用方括号 `[]` 定义字符类，可以匹配其中任意一个字符。例如，`[abc]` 匹配 `a`、`b` 或 `c`。
4. **预定义字符类**：常见的预定义字符类包括 `\d`（数字），`\w`（字母、数字或下划线），`\s`（空白字符），等价于 `[0-9]`、`[a-zA-Z0-9_]`、`[ \t\n\r\f\v]`。
5. **量词**：量词用来指定匹配次数。例如，`*`（0 次或多次），`+`（1 次或多次），`?`（0 次或 1 次），`{n}`（恰好 n 次），`{n,}`（至少 n 次），`{n,m}`（n 到 m 次）。
6. **边界匹配**：边界匹配符包括 `^`（行首），`$`（行尾），`\b`（单词边界），`\B`（非单词边界）。
7. **分组和捕获**：使用圆括号 `()` 将正则表达式的一部分进行分组，可以进行捕获以便后续引用。例如，`(\d{3})-(\d{3})` 匹配 "123-456" 并捕获 "123" 和 "456"。
8. **选择和替代**：使用管道符 `|` 表示选择，匹配左边或右边的表达式。例如，`a|b` 匹配 `a` 或 `b`。

## 工作机制

1. **编译**：正则表达式首先被编译成一种内部表示形式。这个编译过程将解析正则表达式字符串，并将其转换为一个有限状态自动机（Finite State Machine, FSM）。
2. **匹配过程**：
   - **DFA（确定性有限自动机）**：DFA 每个状态都有确定的转换，即每一个输入字符都会导致状态的唯一确定转换。DFA 的匹配过程效率高，但状态数可能较多。
   - **NFA（非确定性有限自动机）**：NFA 允许从一个状态可以有多条转换边，匹配过程可能需要回溯。NFA 的构建比较简单，但匹配过程效率可能较低。
3. **引擎**：
   - **NFA 引擎**：基于回溯算法，尝试每一种可能的路径，直到找到匹配或失败。大多数现代正则表达式引擎（如 Perl、Python、JavaScript 等）使用 NFA 引擎。
   - **DFA 引擎**：不使用回溯，直接通过状态转换进行匹配，通常速度较快，但实现较为复杂且内存消耗较大。

## 应用场景

- **字符串查找和替换**：在文本编辑器、IDE 等工具中，通过正则表达式进行复杂的搜索和替换操作。
- **数据验证**：验证输入数据的格式，如邮箱地址、电话号码等。
- **文本处理**：如日志解析、数据抽取等。

通过对正则表达式的理解和应用，可以高效地进行字符串处理和模式匹配，极大地提高文本处理的灵活性和效率。



![image-20240929153836955](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929153836955-1727921863704-1.png)

JS 正则表达式对象

```js
var regex = new RegExp("^\\w{6,12}$")

var regex = /^\w{6,12}$/

regex.test(str) //正则表达式
```



- `\d`: 数字
- `\D`: 除了数字以外的字符
- `\s`:空白字符
- `\w`:字母+下划线+数字
- `[ ]`匹配单个字符 字符集合
- `[^1-3A-Z]`匹配一个除了1-3和大写字母的字符
- 尖角^只有在方框内部才是取反的意思。在外部表示匹配开头，`^abc`表示作为开头的abc，`BCD$`就是作为结尾的BCD
  - 匹配总共需要三个参数：要处理的字符串，匹配模式和正则表达式本身，可以看出处理的文本本身就是一个字符串。
  - 如果没有多行匹配，^和$只会作用于**整个文本**的开头和结尾
  - multiline模式会匹配每一行
- `/...../g`表示不要搜索一次就返回，去掉只显示第一个匹配的
- `/...../m`表示多行匹配
- `/...../i`表示忽略大小写
- `\b`:表示单词边界 `\bin`表示in在单词开头 `in\b`表示in在单词末尾
- `\B`:表示单词内部 `\Bin\B`表示in在单词中间 `\Bin`表示匹配前面有东西的in
- `.$`表示属于结尾的任意字符
- `\.$`表示属于结尾的点号

### 量词

- `at+` atttt atttt 加号表示a t(重复一次或者多次)
- `at*` atttt atttt 星号表示a t(重复0次或者多次)
- `at?` at a 问号表示零次或者一次
- `at{3,5}` attt 表示t重复了3到5次
- `at{3,}` attt 表示t重复了3次以上 

### 分组

- `(at){3}` at整体重复了3次
- `(P|p)iece` Piece和piece
- `(d{4})[-/_.](d{1,2})[-/_.](d{1,2})` 匹配年月日并且分成三组$1\$2\$3分别代表123组可以使用替换
- 如果不想分其中一个组`?:`表示仅代表一个整体不参与分组

在 Java 的正则表达式中，如果你想忽略大小写匹配，可以使用以下几种方法：

## Java中忽略大小写的使用

### 使用 `Pattern.CASE_INSENSITIVE` 标志

通过调用 `Pattern.compile` 方法并传递 `Pattern.CASE_INSENSITIVE` 标志，可以忽略大小写。

示例：

```java
import java.util.regex.*;

public class Main {
    public static void main(String[] args) {
        String text = "Hello World!";
        String regex = "hello";

        // 编译正则表达式时指定忽略大小写
        Pattern pattern = Pattern.compile(regex, Pattern.CASE_INSENSITIVE);
        Matcher matcher = pattern.matcher(text);

        if (matcher.find()) {
            System.out.println("匹配成功！");
        } else {
            System.out.println("匹配失败！");
        }
    }
}
```

### 使用 `(?i)` 内联标志

在正则表达式的开头加上 `(?i)` 也可以忽略大小写。这是内联标志，不需要使用 `Pattern.compile`。

示例：

```java
public class Main {
    public static void main(String[] args) {
        String text = "Hello World!";
        String regex = "(?i)hello";

        if (text.matches(regex)) {
            System.out.println("匹配成功！");
        } else {
            System.out.println("匹配失败！");
        }
    }
}
```

### 总结

- **`Pattern.CASE_INSENSITIVE`** 标志是在编译正则表达式时通过代码设置忽略大小写。
- **`(?i)`** 是在正则表达式内部直接指定忽略大小写，简单方便。

# [AJAX, JSON](./个人笔记/Java Web 服务端.pdf) 

# Vue

[Vue.js - 渐进式 JavaScript 框架 | Vue.js (vuejs.org)](https://cn.vuejs.org/)

## 优点、原理

- 免除DOM操作，简化书写
- 基于MVVM思想，双向绑定数据，使view能够绑定model的数据，而不是model变了

![image-20240929223353641](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929223353641.png)

model: plain JavaScript Objects     view: 模板

model，也就是数据，通过vue实例绑定到view上，model一更新，view上也会立即更新，

view，模板，通过vue实例的domListener，监听发生的dom事件，比如输入，通过listener将model处的数据实时更新。

数据全存在vm上，view能看到vm所有的数据



![image-20240929231110532](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929231110532.png)

在之前的JSON案例中，填写表单时，表单数据是model，**在提交的时候**，还要用**繁琐的dom操作**获取输入框中的值再赋给model。vue能在输入的时候就赋值给对象，如果从服务器接收到数据也能实时更新。

## 上手

[index.html](.\个人笔记\HTML\index.html)

**Vue3** 

```html
<div id="hello-vue" class="demo">//div元素 唯一id为hello-vue 归属于demo类
<input v-model="message">//v-model双向绑定,这个view中的值是多少, vue实例中数据就是多少
{{ message }}//插值表达式。这是 Vue.js 的模板语法，用于将 Vue 实例中的数据绑定到页面上。
			 //vue实例中的数据变化时,这里也跟着变化。
</div>

<script>
const HelloVueApp={ //此处创建了一个自定义JS对象
	data(){//data是一个无参的函数,返回值是一个对象，里面包含了model数据
		return {
            message: 'Hello,Vue!'//数据初值是Hello,Vue!字符串
        }
	}
}
Vue.createApp(HelloVueApp).mount("#hello-vue")
//Vue.createApp() 方法用于创建一个 Vue 应用实例，参数是一个包含组件选项的对象（这里是 HelloVueApp）。
//.mount('#hello-vue') 方法将 Vue 应用实例挂载到页面中具有 id="hello-vue" 的 DOM 元素上。
</script>

```

## 指令

指令是带有前缀 **v-** 的特殊属性，用于在模板中表达逻辑。

**v-bind**: 动态绑定一个或多个特性，或一个组件 prop。将 Vue 实例的数据绑定到 HTML 元素的属性上

```html
<a v-bind:href="url">Link</a>
```

简写：

```html
<a :href="url">Link</a>
```

**v-if**: 条件渲染。根据表达式的值来条件性地渲染元素或组件：

```html
<p v-if="seen">Now you see me</p>
```

**v-for**: 列表渲染。 用于根据数组或对象的属性值来循环渲染元素或组件。

```html
<ul>
  <li v-for="item in items" :key="item.id">{{ item.text }}</li>
</ul>
//v-for=迭代对象 相当于增强for循环 
//key帮助 Vue 在渲染列表时识别每个元素的唯一性。
<script>
const HelloVueApp = {
  data() {
    return {
      items: [ //items数组包含多个匿名对象，这些匿名对象id和text两个属性
        { id: 1, text: 'Item 1' },
        { id: 2, text: 'Item 2' },
        { id: 3, text: 'Item 3' }
      ]
    }
  }
}

Vue.createApp(HelloVueApp).mount('#app')
</script>
```

**v-model**: 实现表单数据双向绑定：

```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

**v-on**: 事件监听器。 用于在 HTML 元素上绑定事件监听器，使其能够触发 Vue 实例中的方法或函数。

```html
<button v-on:click="doSomething">Click me</button>
```

简写：

```html
<button @click="doSomething">Click me</button>
```

**v-show**: 用于根据表达式的值来条件性地显示或隐藏元素。

```html
<div id="hello-vue" class="demo">
    <button v-on:click="showMessage = !showMessage">显示/隐藏</button>
    <p v-show="showMessage">Hello Vue!</p>
</div>

<script>
const HelloVueApp = {
  data() {
    return {
      showMessage: true
    }
  }
}

Vue.createApp(HelloVueApp).mount('#hello-vue')
</script>
```

### 事件监听

v-on指令能够监听事件

```js
v-on:click="methodName"
@click="methodName"

<div id="app">
  <button @click="counter += 1">增加 1</button>
  <p>这个按钮被点击了 {{ counter }} 次。</p>
</div>
 
<script>
const app = {
  data() {
    return {
      counter: 0
    }
  }
}
 
Vue.createApp(app).mount('#app')
</script>
```

### 案例简化-JavaScript中的this指针与回调函数

```js
<div id="app">
    
    
    </div>

//script
const VueApp = {
    data(){
    return{
        brands:[]
    }
},
    mounted(){
        axios.get("http://localhost:8080/selectAllServlet")
         .then((resp)=>{this.brands = resp.data;});
        //brands是集合,用v-for遍历
        //回调函数是作为参数传递给另一个函数的函数，目的是在特定事件或条件发生时被调用。它通常用于异步操作、事件处理和在特定时机执行代码。箭头函数不会根据调用者创建this，只会寻找域中含有的this指针，mounted的this就是Vue实例
    }
    methods:{
    	show: function(){
            alert("clicked");//跟提交的按钮绑定
        }
	}
}
Vue.createVueApp(VueApp).mount(#app)


```

[JavaScript中的this指针](.\个人笔记\HTML\anoymousThis.html) 另外一种是在mounted中先定义一个var _this = this，这样\_this 一定指代的是Vue实例

```html
<div id="app">
  <p>单个复选框：</p>
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{{ checked }}</label>
    
  <p>多个复选框：</p>
  <input type="checkbox" id="runoob" value="Runoob" v-model="checkedNames">
  <label for="runoob">Runoob</label>
  <input type="checkbox" id="google" value="Google" v-model="checkedNames">
  <label for="google">Google</label>
  <input type="checkbox" id="taobao" value="Taobao" v-model="checkedNames">
  <label for="taobao">taobao</label>
  <br>
  <span>选择的值为: {{ checkedNames }}</span>
</div>
 
<script>
const app = {
  data() {
    return {
      checked : false,
      checkedNames: []
    }
  }
}
 
Vue.createApp(app).mount('#app')
</script>
```

# Element

[Element - 网站快速成型工具](https://element.eleme.cn/#/zh-CN)

Vue组件库，可以快速构建网页

![image-20240930212139910](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240930212139910.png)

## 布局

### Layout

![image-20240930224145033](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240930224145033.png)

### Container

![image-20240930224317166](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240930224317166.png)

## 组件

![image-20240930224538591](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240930224538591.png)

对话框+表单
