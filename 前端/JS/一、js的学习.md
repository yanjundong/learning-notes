# 一、数据类型 

## 1、Map和Set

`Map`和`Set`是ES6标准新增的数据类型

**Map**

`Map` 是一组键值对的结构，具有极快的查找速度。

如果用对象类型来保存学生的成绩

```js
var m = {
  {'name': 'Michael', 'score': 95},
    {'name': 'Bob', 'score': 75},
    {'name': 'Tracy', 'score': 85},
}
```

需要循环遍历查找到名字，然后再获得成绩，如果学生越多，查找的速度越慢。



如果用Map实现，只需要一个“名字”-“成绩”的对照表，直接根据名字查找成绩，无论这个表有多大，查找速度都不会变慢。

```js
var m = new Map([['Michael', 95], ['Bob', 75], ['Tracy', 85]]);
m.get('Michael'); // 95
```

初始化 `Map` 需要一个二维数组，或者直接初始化一个空`Map`。`Map`具有以下方法：

```js
var m = new Map(); // 空Map
m.set('Adam', 67); // 添加新的key-value
m.set('Bob', 59);
m.has('Adam'); // 是否存在key 'Adam': true
m.get('Adam'); // 67
m.delete('Adam'); // 删除key 'Adam'
m.get('Adam'); // undefined
```

> ps: 多次对一个key放入value时，后面的值会把前面的值替换



**Set**

`Set`和`Map`类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在`Set`中，没有重复的key。

创建 `set`：

```js
var s1 = new Set(); // 空Set
var s2 = new Set([1, 2, 3]); // 含1, 2, 3
```

重复元素在`Set`中自动被过滤：

```js
var s = new Set([1, 2, 3, 3, '3']);
s; // Set {1, 2, 3, "3"}
```

> 注意数字`3`和字符串`'3'`是不同的元素。

通过`add(key)`方法可以添加元素到`Set`中，可以重复添加，但不会有效果：

```js
s.add(4);
s; // Set {1, 2, 3, 4}
s.add(4);
s; // 仍然是 Set {1, 2, 3, 4}
```

通过`delete(key)`方法可以删除元素：

```js
var s = new Set([1, 2, 3]);
s; // Set {1, 2, 3}
s.delete(3);
s; // Set {1, 2}
```



## 2、Iterable

遍历`Array`可以采用下标循环，遍历`Map`和`Set`就无法使用下标。为了统一集合类型，ES6标准引入了新的`iterable`类型，`Array`、`Map`和`Set`都属于`iterable`类型。

### 1）、 for ... of

具有`iterable`类型的集合可以通过 `for ... of`循环来遍历。

```js
let a = ['A', 'B', 'C'];
let s = new Set(['A', 'B', 'C']);
let m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
for (let x of a) { // 遍历Array
  console.log(x); //A B C
}
for (let x of s) { // 遍历Set
  console.log(x); //A B C
}
for (let x of m) { // 遍历Map
  console.log(x[0] + '=' + x[1]);	//1=x 2=y 3=z
}
```



**`for ... of`循环和`for ... in`循环的区别?**

`for ... in`循环由于历史遗留问题，它遍历的实际上是对象的属性名称。

一个`Array`数组实际上也是一个对象，它的每个元素的索引被视为一个属性。

```js
let a = ['A', 'B', 'C'];
for (let x in a) {
  console.log(x); // 0 1 2 
}
for (let x of a) {	//for ... of只循环集合本身的元素：
  console.log(x); // A B C
}
```

### 2）、forEach

更好的方式是直接使用`iterable`内置的`forEach`方法，它接收一个函数，每次迭代就自动回调该函数。

```js
let a = ['A', 'B', 'C', 'D'];
a.forEach(function (element, index, array) {
    // element: 指向当前元素的值
    // index: 指向当前索引
    // array: 指向Array对象本身
    console.log(element + ', index = ' + index);
});

```

> ps:  在`Set`中，回调函数的前两个参数都是元素本身。
>
> `Map`的回调函数参数依次为`value`、`key`和`map`本身



# 二、函数

## 1、函数参数

### 1）、typeof

对函数参数类型的检查

```js
//x是否为数值
typeof x !== 'number'
```

### 2）、arguments

`arguments`，只在函数内部起作用，永远指向调用者传入的所有参数。`arguments`类似`Array`但它不是一个`Array`：

```js
foo(x) {
  console.log('x = ' + x); // 10
  for (var i=0; i<arguments.length; i++) {
    console.log('arg ' + i + ' = ' + arguments[i]); // 10, 20, 30
  }
}
foo(10, 20, 30)
//x = 10
//arg 0 = 10
//arg 1 = 20
//arg 2 = 30
```

`arguments`最常用于判断传入参数的个数:

```js
// foo(a[, b], c)
// 接收2~3个参数，b是可选参数，如果只传2个参数，b默认为null：
function foo(a, b, c) {
    if (arguments.length === 2) {
        // 实际拿到的参数是a和b，c为undefined
        c = b; // 把b赋给c
        b = null; // b变为默认值
    }
    console.log(a + ' ' + b + ' ' + c);
}

foo(10, 20)	//10 null 20
```



### 3）、rest参数

ES6标准引入了rest参数，上面的函数可以改写为：

```js
function foo(a, b, ...rest) {
    console.log('a = ' + a);
    console.log('b = ' + b);
    console.log(rest);
}

foo(1, 2, 3, 4, 5);
// 结果:
// a = 1
// b = 2
// Array [ 3, 4, 5 ]

foo(1);
// 结果:
// a = 1
// b = undefined
// Array []	->空数组，不是undefined
```



## 2、高阶函数

一个函数可以接收另一个函数作为参数，这种函数就称之为高阶函数。

### 1）、map

![](https://gitee.com/yanjundong97/picBed/raw/master/images/map函数.png)

`map` 函数可以传入一个函数参数 `map(f(x))`，把f(x)作用在Array的每一个元素并把结果生成一个新的Array。

```js
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
var results = arr.map(function pow(x){
  return x * x;
}); // [1, 4, 9, 16, 25, 36, 49, 64, 81]
console.log(results);
```

可以计算任意复杂的函数，比如，把`Array`的所有数字转为字符串：

```js
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
arr.map(String); // ['1', '2', '3', '4', '5', '6', '7', '8', '9']
```



### 2）、reduce

Array的`reduce()`把一个函数作用在这个`Array`的`[x1, x2, x3...]`上，这个函数必须接收两个参数。

第一次执行时，从Array中取x1、x2，在之后，把执行结果作为第一个参数，从数组中取一个数作为第二个参数。

比方说对一个`Array`求和，就可以用`reduce`实现：

```js
var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x + y;	 
  //1+3
  //4+5
  //9+7
  //16+9
}); // 25
```



### 3）、filter

filter用于把`Array`的某些元素过滤掉，然后返回剩下的元素。

`filter()`也接收一个函数，该函数总是返回 `boolean` 类型。

```js
//把一个Array中的空字符串删掉，可以这么写：
var arr = ['A', '', 'B', null, undefined, 'C', '  '];
var r = arr.filter(function (s) {
    return s && s.trim(); // 注意：IE9以下的版本没有trim()方法
});
r; // ['A', 'B', 'C']
```

> `s.trim()` 删除字符串的头尾空格

```js
//去除数组中的重复元素
let arr = ['apple', 'strawberry', 'banana', 'pear', 'apple', 'orange', 'orange', 'strawberry'];
let r = arr.filter(function (element, index, self) {
  // element: 指向当前元素的值
    // index: 指向当前索引
    // self: 指向arr本身
    return self.indexOf(element) === index;
});

```

> `indexOf`总是返回第一个元素的位置



### 4）、sort

JavaScript的`Array`的`sort()`方法就是用于排序的，但是字符串排序时根据ASCII码进行排序，在数字的排序时也是先把转化为String类型在排序。

```js
// 看上去正常的结果:
['Google', 'Apple', 'Microsoft'].sort(); // ['Apple', 'Google', 'Microsoft'];

// apple排在了最后:
['Google', 'apple', 'Microsoft'].sort(); // ['Google', 'Microsoft", 'apple']

// 数值的错误排序:
[10, 20, 1, 2].sort(); // [1, 10, 2, 20]
```



`sort()`方法也是一个高阶函数，它还可以接收一个比较函数来实现自定义的排序：

```js
arr.sort(function (x, y) {
    if (x < y) {
        return -1;
    }
    if (x > y) {
        return 1;
    }
    return 0;
});
console.log(arr); // [1, 2, 10, 20]
```

> `sort()`方法会直接对`Array`进行修改，它返回的结果仍是当前`Array`：



### 5）、Array

`Array`对象还提供了很多非常实用的高阶函数

#### every

`every()`方法可以判断数组的所有元素是否满足测试条件。

```js
var arr = ['Apple', 'pear', 'orange'];
console.log(arr.every(function (s) {
    return s.length > 0;
})); // true, 因为每个元素都满足s.length>0

console.log(arr.every(function (s) {
    return s.toLowerCase() === s;
})); // false, 因为不是每个元素都全部是小写
```



#### find

`find()`方法用于查找符合条件的第一个元素，如果找到了，返回这个元素，否则，返回`undefined`：

```js
var arr = ['Apple', 'pear', 'orange'];
console.log(arr.find(function (s) {
    return s.toLowerCase() === s;
})); // 'pear', 因为pear全部是小写

console.log(arr.find(function (s) {
    return s.toUpperCase() === s;
})); // undefined, 因为没有全部是大写的元素
```



#### findIndex

`findIndex()`和`find()`类似，也是查找符合条件的第一个元素，不同之处在于`findIndex()`会返回这个元素的索引，如果没有找到，返回`-1`



#### forEach

`forEach()`常用于遍历数组，因此，传入的函数不需要返回值，



## 3、闭包



## 4、箭头函数

```js
x => x * x
```

等同于

```js
function(x) {
  return x * x;
}
```

箭头函数相当于匿名函数，并且简化了函数定义。箭头函数有两种格式:

- 一种像上面的，只包含一个表达式，连`{ ... }`和`return`都省略掉了
- 还有一种可以包含多条语句，这时候就不能省略`{ ... }`和`return`：

如果参数不止一个，就需要用括号`()`括起来：

```js
// 两个参数:
(x, y) => x * x + y * y

// 无参数:
() => 3.14
```

> 如果要返回一个对象的话，必要要写成
>
> ```js
> x => ({ foo: x })
> ```



## 5、generator

generator（生成器）是ES6标准引入的新的数据类型。一个generator看上去像一个函数，但可以返回多次。

`function` 定义的函数只能返回一次，如果换成 `generator` ，就可以一次返回一个数，不断返回多次。

```js
function* fib(max) {
    let
        t,
        a = 0,
        b = 1,
        n = 0;
    while (n < max) {
        yield a;
        [a, b] = [b, a + b];
        n ++;
    }
    return;
}
```

直接调用试试：

```js
fib(5);
```

直接调用一个generator和调用函数不一样，`fib(5)`仅仅是创建了一个generator对象，还没有去执行它。

调用generator对象有两个方法

- 不断地调用generator对象的`next()`方法：

  ```js
  let f = fib(5);
  f.next(); // {value: 0, done: false}
  f.next(); // {value: 1, done: false}
  f.next(); // {value: 1, done: false}
  f.next(); // {value: 2, done: false}
  f.next(); // {value: 3, done: false}
  f.next(); // {value: undefined, done: true}
  //next()方法会执行generator的代码，然后，每次遇到yield x;就返回一个对象{value: x, done: true/false}，然后“暂停”。返回的value就是yield的返回值，done表示这个generator是否已经执行结束了。如果done为true，则value就是return的返回值。
  ```

- 直接用`for ... of`循环迭代generator对象

  ```js
  for (let x of fib(10)) {
      console.log(x); // 依次输出0, 1, 1, 2, 3, 5, ...
    }
  ```



**`generator`和普通函数相比，有什么用？**

因为`generator`在执行时遇到`yield x`就会暂停，所以它看上去就像一个可以记住执行状态的函数。

例如：要生成一个自增的ID，可以编写一个`next_id()`函数：

```js
//由于函数无法保存状态，故需要一个全局变量`current_id`来保存数字。
var current_id = 0;

function next_id() {
    current_id ++;
    return current_id;
}
```

如果用`generator`来实现：

```js
function* fib() {
  let current_id = 0;
  while(true) {
    current_id ++;
    yield current_id;
  }
  return;
}
```

调用`generator`

```js
let f = fib();
console.log(f.next());; // {value: 1, done: false}
console.log(f.next());; // {value: 2, done: false}
console.log(f.next());; // {value: 3, done: false}
console.log(f.next());; // {value: 4, done: false}
console.log(f.next());; // {value: 5, done: false}
console.log(f.next());; // {value: 6, done: false}
```



 # 三、标准对象

`typeof`操作符获取对象的类型，它总是返回一个字符串：

```js
typeof 123; // 'number'
typeof NaN; // 'number'
typeof 'str'; // 'string'
typeof true; // 'boolean'
typeof undefined; // 'undefined'
typeof Math.abs; // 'function'
typeof null; // 'object'
typeof []; // 'object'
typeof {}; // 'object'
```

**包装对象**

`number`、`boolean`和`string`都有包装对象，类似于Java中的`int` 和 `Integer`。在JavaScript中，字符串也区分`string`类型和它的包装类型。包装对象用`new`创建：

```js
var n = new Number(123); // 123,生成了新的包装类型
var b = new Boolean(true); // true,生成了新的包装类型
var s = new String('str'); // 'str',生成了新的包装类型
```

虽然包装对象看上去和原来的值一模一样，显示出来也是一模一样，但他们的类型已经变为`object`了！所以，包装对象和原始值用`===`比较会返回`false`：

```js
typeof new Number(123); // 'object'
new Number(123) === 123; // false

typeof new Boolean(true); // 'object'
new Boolean(true) === true; // false

typeof new String('str'); // 'object'
new String('str') === 'str'; // false
```

## \== 和 ===的区别

==：称为等值符，先转化为类型相同的值，再进行比较；

===：称为等同符，当两边值的类型相同时，直接比较值，若类型不相同，直接返回false；



如果在使用`Number`、`Boolean`和`String`时，没有写`new`，此时，`Number()`、`Boolean`和`String()`被当做普通函数，把任何类型的数据转换为`number`、`boolean`和`string`类型

```js
var n = Number('123'); // 123，相当于parseInt()或parseFloat()
typeof n; // 'number'

var b = Boolean('true'); // true
typeof b; // 'boolean'

var b2 = Boolean('false'); // true! 'false'字符串转换结果为true！因为它是非空字符串！
var b3 = Boolean(''); // false

var s = String(123.45); // '123.45'
typeof s; // 'string'
```



## 1、总结

- 不要使用`new Number()`、`new Boolean()`、`new String()`创建包装对象；
- 用`parseInt()`或`parseFloat()`来转换任意类型到`number`；
- 用`String()`来转换任意类型到`string`，或者直接调用某个对象的`toString()`方法；
- 通常不必把任意类型转换为`boolean`再判断，因为可以直接写`if (myVar) {...}`；
- `typeof`操作符可以判断出`number`、`boolean`、`string`、`function`和`undefined`；
- 判断`Array`要使用`Array.isArray(arr)`；
- 判断`null`请使用`myVar === null`；
- 判断某个全局变量是否存在用`typeof window.myVar === 'undefined'`；
- 函数内部判断某个变量是否存在用`typeof myVar === 'undefined'`。

**注意：**

`number`对象调用`toString()`报SyntaxError：

```js
123.toString(); // SyntaxError
```

遇到这种情况，要特殊处理一下：

```js
123..toString(); // '123', 注意是两个点！
(123).toString(); // '123'
```



## 2、Date

要获取系统当前时间，用：

```js
var now = new Date();
now; // Thu Feb 20 2020 12:59:27 GMT+0800 (中国标准时间)
now.getFullYear(); // 2020, 年份
now.getMonth(); // 1, 月份，注意月份范围是0~11，5表示六月
now.getDate(); // 20, 表示24号
now.getDay(); // 4 表示星期四
now.getHours(); // 12, 24小时制
now.getMinutes(); // 59, 分钟
now.getSeconds(); // 27, 秒
now.getMilliseconds(); // 983, 毫秒数
now.getTime(); // 1582174767983, 以number形式表示的时间戳
```

> 当前时间是浏览器从本机操作系统获取的时间，所以不一定准确，因为用户可以把当前时间设定为任何值



## 3、RegExp

正则表达式：凡是符合规则的字符串，我们就认为它“匹配”了，否则，该字符串就是不合法的。

### 1）、规则

1. `\d`可以匹配一个数字，`\w`可以匹配一个字母或数字，`\s`可以匹配一个空格。

2. 要匹配变长的字符，用`*`表示任意个字符（包括0个），用`+`表示至少一个字符，用`?`表示0个或1个字符，用`{n}`表示n个字符，用`{n,m}`表示n-m个字符：

3. 要做更精确地匹配，可以用`[]`表示范围
4. `A|B`可以匹配A或B
5. `^`表示行的开头，`^\d`表示必须以数字开头

6. `$`表示行的结束，`\d$`表示必须以数字结束



JavaScript有两种方式创建一个正则表达式：

- 直接通过`/正则表达式/`写出来
- 通过`new RegExp('正则表达式')`创建一个RegExp对象。

```js
var re1 = /ABC\-001/;
var re2 = new RegExp('ABC\\-001');	//因为字符串的转义问题，字符串的两个\\实际上是一个\

re1; // /ABC\-001/
re2; // /ABC\-001/
```

### 2）、test

`test()`方法用于测试给定的字符串是否符合条件：

```js
var re = /^\d{3}\-\d{3,8}$/;
re.test('010-12345'); // true
re.test('010-1234x'); // false
re.test('010 12345'); // false
```



### 3）、切分字符串

用正则表达式切分字符串比用固定的字符更灵活，请看正常的切分代码：

```js
'a b   c'.split(' '); // ['a', 'b', '', '', 'c']
```

无法识别连续的空格，用正则表达式试试：

```js
'a b   c'.split(/\s+/); // ['a', 'b', 'c']
```

无论多少个空格都可以正常分割。加入`,`试试：

```js
'a,b, c  d'.split(/[\s\,]+/); // ['a', 'b', 'c', 'd']
```



### 4）、分组

正则表达式还有提取子串的强大功能。用`()`表示的就是要提取的分组（Group）。

```js
var re = /^(\d{3})-(\d{3,8})$/;
re.exec('010-12345'); // ['010-12345', '010', '12345']
re.exec('010 12345'); // null
```

如果正则表达式中定义了组，就可以在`RegExp`对象上用`exec()`方法提取出子串来。

`exec()`方法在匹配成功后，会返回一个`Array`，第一个元素是正则表达式匹配到的整个字符串，后面的字符串表示匹配成功的子串。

`exec()`方法在匹配失败时返回`null`。



## 4、JSON

**序列化**

用`JSON.stringify()`把JavaScript对象序列化成JSON格式的字符串。

**反序列化**

用`JSON.parse()`把JSON格式的字符串变成一个JavaScript对象



# 四、浏览器

## 1、浏览器对象





## 2、Promise

异步操作会在将来的某个时间点触发一个函数调用。





