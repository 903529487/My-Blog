
### 第一章： 作用域是什么

#### 1、 编译原理

JavaScript 被列为 ‘动态’ 或 ‘解释执行’ 语言，于其他传统语言（如 java）不同的是，JavaScript是边编译边执行的。
一段源码在执行前会经历三个步骤： `分词/词法分析` -> `解析/语法分析` -> `代码生成`

  - 分词/词法分析
  这个过程将字符串分解成词法单元，如 var a = 2; 会被分解成词法单元 var、 a、 = 、2、;。空格一般没意义会被忽略

  - 解析/语法分析
  这个过程会将词法单元转换成‘抽象语法树’（Abstract Syntax Tree,AST）。
  如  var a = 2; 对应的 抽象语法树 如下, 可通过 [在线可视化AST](https://astexplorer.net/) 网址去分析

```
{
  "type": "Program",
  "start": 0,
  "end": 10,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 10,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 9,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 5,
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "start": 8,
            "end": 9,
            "value": 2,
            "raw": "2"
          }
        }
      ],
      "kind": "var"
    }
  ],
  "sourceType": "module"
}
```
  - 代码生成
  将 AST 转换成可执行的代码，存放于内存中，分配内存和转化为一些机器指令

#### 2、理解作用域

其实结合上面提到的编译原理，作用域就好理解了。作用域就是当前执行代码对这些标识符的访问权限。
编译器会在当前作用域中声明一个变量，运行时引擎会去作用域中查找该变量（其实就是一个寻址的过程），
如果找到该变量就可以操作变量，找不到就往上一层作用域找（作用域链的概念），或者返回 null


### 第三章： 函数作用域和块作用域

#### 1、函数中的作用域

每声明一个函数都会形成一个作用域，那这个作用域有什么用呢，它能让该作用域内的变量有函数不被外界访问到，也可以反过来说是不让该作用域内的变量或函数污染全局。
对比：
```
var a = 123
function bar() {
  //...
}
```
和
```
function foo() {
  var a = 123
  function bar() {
    //...
  }
}
```
咱们将变量 a 和函数 bar 用一个函数 foo 给包裹起来了，函数 foo 也就形成了一个作用域，变量 a 和函数 bar 外界将不能访问到，同时变量或函数也不会污染全局。

#### 2、函数作用域

那可以你会进一步思考，上面例子的变量 a 和函数 bar 是有了作用域了，但函数 foo 不也是暴露再来全局，也对全局造成污染了啊。是的，JavaScript对这种情况提出了解决方案，有时候咱们在写没有模块化的原生代码也会用到：`立即执行函数 (IIFE)`

```
(function foo() {
  var a = 123
  function bar() {
    //...
  }
})()
```

第一个（）将函数变成表达式，第二个（）执行了这个函数，最终函数 foo 也形成了自己的作用域，不会污染到全局，同时也不被全局访问的到。

#### 3、块作用域

es6之前JavaScript是没有块作用域这个概念了，这与一般的语言（如java ，c）很大不同，当时我理解到这个概念时也是吃惊。看下面这个例子：

```
for (var i = 0; i < 10; i++) {
  console.log('i=', i);
}
console.log('输出', i); // 输出 10
```
for 循环定义了变量 i，通常我们只想这个变量 i 只在循环内使用，但忽略了 i 其实是作用在外部作用域(函数或全局)的。所以循环过后我们也能正常的打印出 i 。

再多提一点，甚至连 try/catch 也没形成块作用域:

```
try {
  for (var i = 0; i < 10; i++) {
    console.log('i=', i);
  }
} catch (error) {}
console.log('输出', i); // 输出 10
```

> 解决方法1

解决块作用域的方法当然是使用 es6 的 let 和 const 了， let 为其声明的变量隐式的劫持了所在的块作用域。

```
for (let i = 0; i < 10; i++) {
  console.log('i=', i);
}
console.log('输出', i); // ReferenceError: i is not defined
```
我们将上面例子的 var 换成 let 最后输出就报错了 ReferenceError: i is not defined ，说明被 let 声明的 i 只作用在了 for 这个块中。

除了 let 会让 for、if、try/catch 等形成块，其实 JavaScript 的 `{}` 也能形成块

```
{
let name = '曾田生'
}

console.log(name); //ReferenceError: name is not defined
```

> 解决方法2

早在没 es6 的 let 声明之前，我们常用的做法是利用函数也能形成作用域这么个概念来解决一些问题的。

看个例子
```
function foo() {
  var result = []
  for (var i = 0; i < 10; i++) {
    result[i] = function () {
      return i
    }
  }
  console.log(i）// i 作用在整个函数，for 执行完此时 i 已经等于 10 了
  return result
}
var result = foo()
console.log(result[0]()); // 10
console.log(result[1]()); // 10
console.log(result[2]()); // 10
```

最后打印的log可看出执行数组函数最终都输出了 10， 因为i 作用在整个函数，for 执行完此时 i 已经等于 10 了, 所以当后续执行函数 `result[x]()` 内部返回的 i 已经是 10 了。

我们可以利用函数的作用域来解决

```
function foo() {
  var result = []
  for (var i = 0; i < 10; i++) {
    result[i] = function (num) {
      return function () { // 函数形成一个作用域，内部变量被私有化了
        return num
      }
    }(i)
  }
  return result
}
var result = foo()
console.log(result[0]()); // 0
console.log(result[1]()); // 1
console.log(result[2]()); // 2
```

上面的例子也是挺典型的，一般面试题比较考基础的话就会被问道，上面例子不仅考察到了块作用域的概念，函数作用域的概念，还考察到了闭包的概念（闭包后续将但不影响这个例子的理解），多琢磨一下就理解了。

