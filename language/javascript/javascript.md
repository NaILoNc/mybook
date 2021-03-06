# 参考规范
  airbnb [https://github.com/airbnb/javascript](https://github.com/airbnb/javascript)

# 闭包
```javascript
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + i;
        result.push( function() {console.log(item + ' ' + list[i]+''+i)} );
    }
    return result;
}

function testList() {
    var fnlist = buildList([1,2,3]);
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}
```
输出


    item2 undefined 3
    item2 undefined 3
    item2 undefined 3

因为数组并没有下标为3的项,所以undefined
那么为什么是item2 和 3 呢
首先var i 作用域为在buildList函数内(根据作用域提升)
但输出为`item2`
可见`for`循环的确执行到` i = 2`时就跳出了
但是在整个作用域内`i` 运算到了3（`buildList`）
所以闭包内的i为3
这是闭包的特性之一
```javascript
for (let i = 0; i < list.length; i++) {
	let item = 'item' + i;
	result.push( function() {console.log(item + ' ' + list[i]+''+i)} 	);
}
```
将var换成let，因为let是块级作用域，每次for循环中let是独立的，
输出

    item0 1 0
    item1 2 1
    item2 3 2


item同理


# Generator

generator函数的意义在于同步的写异步代码
和yield搭配使用

generator函数我们也称之为遍历器
- 使用方法
 
 ```javascript
var p = 1;
function b(){
    console.log('p:' + p);
}
function* a(){
    yield setTimeout({
        p = 2;
    },100);
    b();
}
```

```javascript
a.next();// {value: 'p:2', done: false} 
a.next();// {value: undefined done: true}
```

>我们都知道setTimout是异步的，如果不使用yield，那么会输出`p:1`
>通过next()方法来移动遍历器内的指针到下一个位置(带yield的语句)
>其结构可以看成是这样:

>>HEAD
>>yield 1
>>yield 2
>>...
>>END //log undefined

>done true表示遍历器已全部运行完毕

- 写法
  由于ES6并没有规定*号的位置
  以下四种写法都是可行的

```javascript
  function * foo(x,y){}
  function *foo(x,y){}
  function* foo(x,y){}
  function*foo(x,y){}
  ```

  为了遍历器执行起来更简单
  我们需要自定义一个执行器

```javascript
  function run(fn){
      let gen = fn();
      function next(err,data){
          let result = gen.next(data);
          if(result.done) return;
          result.value(next);
      }
      next();
  }
  
  run(gen);
  ```



  # async

  async 函数是 generator函数的语法糖

  ```javascript
  async function a(){}
  ```
  等同于
  ```javascript
  function a(){
      return run(function *(){
          ...
      })
  }
  ```
  `run`函数为自定义执行器

  ```javascript
  function run(genF){
      return new Promise(function(resolve, reject){
          let gen = genF();
          function step(nextF){
              try{
                  let next = nextF();
              }catch(e){
                  return reject(e);
              }
              if(next.done){
                  return resolve(next.value);
              }
              Promise.resolve(next.value).then(function(v){
                  step(function(){
                      return gen.next(v);
                  })
              },function(e){
                  step(function{
                      return gen.throw(e);
                  });
              });
          }
          step(function(){
              return gen.next(undefined);
          });
      });
  }
  ```


# function  
- 对比下列两个函数`functionOne`和`functionTwo`

```javascript
functionOne();

var functionOne = function() {
  console.log("Hello!");
};                             


functionTwo();

function functionTwo() {
  console.log("Hello!");
}
```
区别在于
- `functionOne`只在执行到它所在行时定义
- `functionTwo`在包含它的script脚本或函数执行时候时定义
	意味着你不能这样
	```javascript
  let test = true;
  function doSomething(): void {}
		if (test) {
   		function functionThree() { doSomething(); }
	}
	```
	`functionThree`的定义与test无关

# 代码风格

- 要获取返回数组的元素

```javascript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

- 如果你只需要传入只读的对象，不对他进行操作, 可以这样

```javascript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```

- 用对象展开代替`Object.assign`

```javascript
// very bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); // this mutates `original` ಠ_ಠ

// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

// good
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
```
# 注释风格
- 注释和前一行要留空，如果是第一行可以不空行

```javascript
// bad
const active = true;  // is current tab

// good
// is current tab
const active = true;

// bad
function getType() {
  console.log('fetching type...');
  // set the default type to 'no type'
  const type = this.type || 'no type';

  return type;
}

// good
function getType() {
  console.log('fetching type...');

  // set the default type to 'no type'
  const type = this.type || 'no type';

  return type;
}

// also good
function getType() {
  // set the default type to 'no type'
  const type = this.type || 'no type';

  return type;
}
```

# Web Compoent

1. 对Web Compoent的介绍 [http://javascript.ruanyifeng.com/htmlapi/webcomponents.html#toc7](http://javascript.ruanyifeng.com/htmlapi/webcomponents.html#toc7)

# 数组操作

# 值类型和引用类型

值类型 `string` `number`
引用类型 `object` `array`

# 原型链

# 事件捕获和冒泡(capturing and bubble)

当IE < 9 时
浏览器只支持事件冒泡

当IE >= 9 
浏览器支持事件捕获`capture`
```javascript
document.addEventListener('click', function() {}, true)
```

一个简单的示例
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <script>
        function capturing() {
            console.log('capturing');
        }

        function bubble() {
            console.log('bubble');
        }
    </script>
    <style>
        .a {
            height: 100px;
            width: 100px;
            background: #000;
        }

        .b {
            margin: auto;
            height: 50px;
            width: 50px;
            line-height: 100px;
            background: #fff;
        }
    </style>
    <div id="a" class="a" onclick="capturing()">
        <div id="b" class="b" onclick="bubble()"></div>
    </div>
</body>
</html>
```