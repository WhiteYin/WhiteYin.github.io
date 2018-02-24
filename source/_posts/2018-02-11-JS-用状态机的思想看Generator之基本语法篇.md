---
title: JS 用状态机的思想看Generator之基本语法篇
comments: true
toc: true
date: 2018-02-11 18:11:34
tags:
- JavaScript
categories:
- JavaScript
description: 用状态机的思想看Generator的状态跳转和状态间传值
---
# 前言
最近学习了阮一峰老师的[《ECMAScript 6 入门》](http://es6.ruanyifeng.com/)里的Generator相关知识，以及《你不知道的JS》中卷的异步编程部分。同时在SegmentFault问答区看到了一些前端朋友对Generator的语法和执行过程有一些疑问，于是我想分享一下自己对Generator的理解，也许对前端社区会有所帮助。

# Generator本质
Generator的本质是一个状态机，yield关键字的作用是分割两个状态，右边的语句执行在前一个状态，而左边的语句是下一个状态要执行的。如果右边为空则默认为undefined，左边为空默认为一个赋值语句，被赋值的变量永远不会被调用。当调用Generator函数获取一个迭代器时，状态机处于初态。迭代器调用next方法后，向下一个状态跳转，然后执行该状态的代码。当遇到return或最后一个yield时，进入终态。终态的标识就是next方法返回对象的done属性。

# Generator状态跳转
Generator函数执行后会生出一个迭代器，包含3个主要方法：next、throw和return。它们的本质都是改变状态机的状态，但throw和return属于强制改变，next则是按照定义好的流程去改变。下面我来分别讲讲这三种方法。

## next方法
先看下面这个例子：
```
function* gen(){
    console.log("state1");
    let state1 = yield "state1";
    console.log("state2");
    let state2 = yield "state2";
    console.log("end");
}
```
我们声明了一个名为gen的Generator函数，其中有2个yield语句，我们可以归纳出4个状态：
1. 初态：这个状态是gen这个“状态机”的初始状态，什么也不会做；
2. 状态一：初态的下一个状态，跳转到这个状态后执行
    ```
    console.log("state1");
    yield "state1";
    ```
3. 状态二：这个状态会先接收上一个状态传来的数据data，然后执行
    ```
    let state1 = data;
    console.log("state2");
    yield "state2";//注意，这里是最后一个yield
    ```
    这里的data是对上一个状态中`yield "state1"`的替换。
4. 状态三（终态）：因为gen已经执行过最后一个yield表达式，所以状态三也就是状态机的终态。这个状态也接受了上一个状态传来的数据data，执行了
    ```
    let state2 = data;
    console.log("end");
    ```
    同时，还将迭代器返回的对象done属性修改为true，比如`{value:undefined,done:true}`。这代表gen这个状态机已经执行到了终态。  
  
将gen这个Generator函数转换成状态机以后，我们可以在脑中想象出下面这张图：  
![demo](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/demo%E4%B8%80.png)
接下来我们就根据这张图分析下状态间是如何跳转的。  

首先是初态，当Generator函数被执行后，状态机就自动处于初态了。这个状态并不会执行任何语句。  
也就是执行语句：
```
let g = gen();
```
会有一个箭头指向初态，如下图：  
![初始化](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/%E5%88%9D%E5%A7%8B%E5%8C%96.png)  
然后是非初态间的状态跳转。如果你想要按照gen里定义好的状态顺序跳转，那你应该使用next()方法。比如我们第一次执行`g.next()`，gen这个状态机会从初态跳转到状态一。然后再执行`g.next()`，则状态一会向状态二跳转，并且发送数据`undefined`，这是因为next函数没有传参，默认为`undefined`。关于状态间如何传递数据我将在下一节讲。  

当我们不断调用next方法，gen会按照定义好的流程进行状态跳转。而且即使是到了终态，next也会返回对象，只是这个对象的值一直是`{value:undefined,done:true}`。听上去像是在终态后面又新增了一个状态，所以next方法能够不断执行。但是我觉得为了符合状态机的设定，还是将第一个done为true的状态叫做终态比较好。

## return方法
与按部就班的next方法不同，return方法会打破原有的状态序列，并根据开发者的需要跳转到一个新的状态，而这个状态有两个特点：
1. 不是原有状态序列中的任何一个状态；
2. 该状态返回的对象的done属性值为true。
  
  我们继续用上面的例子。如果从状态一跳转到状态二，使用的代码是`g.return();`而不是`g.next()`，那么状态图会变成下面这个样子：  
![使用return方法](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/return.png)  
从图中可以看出，return的行为就是新增一个新·状态二插入在状态一后面，然后从状态一跳转到新·状态二，同时输出`{value:undefined,done:true}`。同样，这里的`undefined`也是因为return方法没有传参。

如果Generator函数里有一个try...finally语句，return新建的状态会插入在执行finally块最后一行语句的状态之后。可以看看[这一节](http://es6.ruanyifeng.com/#docs/generator#Generator-prototype-return)阮一峰老师举的例子。

## throw方法
我喜欢将throw方法当作next和return方法的结合。throw()方法与throw关键字很像，都是抛出一个错误。而Generator函数会根据是否定义捕获语句来进行状态跳转。一共有下面3种情况：
1. 没有try...catch；
2. 下一个状态要执行的语句在try...catch中；
3. throw()方法在一个try...catch中被调用。

### 没有try...catch
继续使用上一章的代码，假设从状态一到状态二使用的是`g.throw()`。
```
function* gen(){
    console.log("state1");
    let state1 = yield "state1";
    console.log("state2");
    let state2 = yield "state2";
    console.log("end");
}
let g = gen();
g.next();
g.throw();
```
首先，状态二的代码`console.log("state2");...`并不在try...catch块中，而且也不是在try...catch块中调用`g.throw()`。那么最后的状态图应该是下面这样：  
![throw()方法](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/throw.png)  
看上去就像是调用了return方法，新增一个状态，同时将输出的对象done属性设置为true。但是有一点不同的是这个对象并不会输出，而是报错：`Uncaught undefined`，因为程序因错误而中断。同样，原本要输出的字符串`state2`也不会输出。

这里我认为需要重视的一个问题是错误是在状态二中的哪一条语句抛出的？修改了代码位置后，我发现throw()方法是将`yield "state1"`替换成`throw undefined`，所以之后的`let state1...`等语句都不会执行。
### 下一个状态在try...catch中
修改上一章的示例代码：
```
function* gen(){
    console.log("state1");
    try{
        let state1 = yield "state1";
        console.log("state2");
    }catch(e){
        console.log("catch it");
    }
    let state2 = yield "state2";
    console.log("end");
}
let g = gen();
g.next();
g.throw();
```
由于状态二要执行的代码被try...catch包裹，所以throw()抛出的错误被catch块捕获，从而程序直接转入catch块执行语句，打印“catch it”。这与JS的错误捕获机制一致，状态图总体并不会变化，只是状态二节点下的执行语句有变化。  
![内部捕获](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/catch.png)  
注意红色圈内的语句，相比较与调用next方法时的状态二，删除了try块中错误抛出位置后的`let state1 = data;console.log("state2");`，添加了catch块中要执行的`console.log("catch it");`，如果有finally块也会把里面的语句添加进去。之后再调用next方法，仍然会按照规定好的流程进行跳转。

这一次，throw方法对状态机的操作与next方法大体相同。但因为他本质上是抛出错误，所以会对程序的代码执行顺序有一定的影响。

### throw()方法在一个try...catch中被调用

只要结合上面2种情况，记住3个规则就行：
1. Genereator内部没有try...catch则当作正常抛出错误处理；
2. 下一个状态在try...catch中时，throw()方法抛出的错误会被捕获，那相当于外部没有捕获错误，与第二种情况一致。
3. 规则2中错误捕获后的状态执行代码报错，按规则1处理。

  这里，针对规则3做一个讲解。  
  
看下面这个例子：
```
function* gen(){
    console.log("state1");
    try{
        let state1 = yield "state1";
        console.log("state2");
    }catch(e){
        err = a;//错误
        console.log("内部捕获");
    }
    let state2 = yield "state2";
    console.log("end");
}
let g = gen();
g.next();
try{
    g.throw();
}catch(e){
    console.log("外部捕获");
}
```
那么原本符合规则2的代码在捕获throw()抛出的错误后又因为没有声明标识符`a`报错，从而被外层catch块捕获。导致看上去就像规则1一样。

## 状态间传值

next、throw和return方法除了状态跳转外，还有一个功能就是为前后两个状态传值。但是它们3个的表现又各不相同。

next给状态传值的表现中规中矩，看看下面的代码：
```
function* gen(){
    let value = yield "你好";
    console.log(value);
}
let g = gen();
g.next();
g.next("再见");
```
当我们想要跳转到执行`console.log(value);`的状态二时，给next方法传一个字符串“再见”，然后`yield "你好"`会被替换成`"再见"`，赋值给value变量打印出来。你可以试试不传值或者传其他值，应该能帮助你理解更深刻。

throw方法一般都会传值，而且为了规范应该传一个Error对象。

return方法传值有点特殊，修改上面的代码：
```
function* gen(){
    let value = yield "你好";
    console.log(value);
}
let g = gen();
g.next();
g.return("看得见我吗？");
```
如果你前面的知识没忘的话，你应该知道，用return替换next后，什么也不会打印。因为跳转到了一个什么代码也不会执行状态。那么return函数的参数作用体现在哪呢？还记得每一个方法调用后都会返回一个对象吗？上面的代码输出了`{value:"看得见我吗",done:true}`。哈，我看见你了。

# 关于终态

一般我喜欢把最后一个yield或是return表达式当作最后一个状态。但是有时候可以把终态想象成一个不断循环自身的状态，比如下面这样：  
![终态](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/%E7%BB%88%E6%80%81.png)  
这样理解有一个好处是可以解释为什么done属性值为true后，再次调用next仍会返回一个对象`{value:undefined,done:true}`。但是这样会多一个状态，画图不方便（假装这个理由很充分）。  
总之，如何理解全看个人喜欢。

# 实际案例
下面利用状态机的思想讲讲两个实际案例。
## 一个小问题
我之前回答过一个[问题](https://segmentfault.com/q/1010000013217614/a-1020000013218371)，把它当作实例来分析一下吧  

题主不太理解下面代码的执行顺序：
```
function* bar() {
  console.log('one');
  console.log('two');
  console.log('three');
  yield console.log('test');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}
let barObj = bar();
barObj.next();
barObj.next('a');
barObj.next('b');
```
让我们来帮他分析分析吧。

首先，我补全了这段代码。
```
function* bar() {
  console.log('one');
  console.log('two');
  console.log('three');
  yield console.log('test');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}
let barObj = bar();
barObj.next();
barObj.next('a');
barObj.next('b');
barObj.next('c');
barObj.next();
```
然后，分析bar这个Genereator声明了几个状态。一共有6个状态，状态图如下：  
![例子](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/demo2.png)   
根据状态图，题主提出的两个问题：
1. 第一次 next 的时候应该走到了 yield console.log('test')
2. 第二次传了一个 a 这个时候程序似乎没有执行  

  第一个问题，调用next方法后，跳转到state1，而`yield console.log('test')`是在state1里执行的，所以确实走到了这行代码。  
然后，调用next("a")，跳转到state2，这里并没有值接收字符串`"a"`，所以自然没有打印出来，造成程序没有执行的假象。

这个问题比较简单，状态图一画就能理解了。

## throw方法的一个特性
第二个实例是我在看《ECMAScript 6 入门》时，阮一峰老师说：
> throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。

然后举了一个例子：
```js
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```
这里我觉得很奇怪，因为按照我的想法，这是显然的呀，为什么要单独说呢？按照我在Generator状态跳转那一章说的，这属于**下一个状态在try...catch中**的情况，因为
```
try{
    /*state2*/yield console.log('a');
}
```
中yield的左侧是state2状态的代码，虽然没有写，但是我们默认为向一个永远不会被调用的变量进行赋值。  
接着是画状态图：  
![demo3](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-11/demo3.png)  
我们只关心`g.throw()`，所以画部分状态图就够了。从图中可以看出，throw方法被调用后，因为错误被捕获，所以正常跳转到了state2，然后必然会执行`yield console.log('b');`。  

# 总结
状态机的知识还是在大学的编译原理课学习的，有些概念已经忘了。不过在看Generator时，我突然觉得用状态机来解释代码的冻结和执行非常直观。只要能够画出相应的状态图就可以知道每一次调用next等方法会执行什么样的代码。靠着状态机的思想，我在学习Generator时基本没有疑惑，所以决定整理并分享出来。  
但是我有点不自信，因为网上搜索了很多次，除了阮一峰老师，并没有人同时提到状态机和Generator两个关键字。我在写这篇文章的时候也偶尔怀疑是不是我错了。不过既然已经写了这么多，而且从我自身感觉以及解决了文中两个例子的情况来看，分享出来让大家指指错也是不错的。    所以，如果有什么问题希望能够在评论中指出。非常感谢你的阅读，祝你新年快乐！