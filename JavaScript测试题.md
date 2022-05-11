## 题1：JavaScript的数据类型

### 问：JS有哪些数据类型？

### 答：

JavaScript一共有6种简单（原始）数据类型，分别是`boolean`、`null`、 `undefined`、 `number`、 `string`、 `symbol` 、然后还要一种特殊的基本数据类型 `bigint`。一种引用数据

一种复杂类型`Object`，但`Object`可以细分为`Function`，`Array`， 普通对象`Object`。

### 追问：JS有哪些常用的内置对象

### 答：

常用的有`Number`、`BigInt`、 `Math`、`Date`、`String`、`RegExp`、`Map`等

### 追问：number和bigInt的区别

### 答：

number最大只可以表示`2^53 - 1 `，bigInt可以表示大于这个数的值。number和bigInt可以弱等，但不能强等，两者可以通过比较符来进行比较。对bigInt的计算都会返回一个bigInt，且bigInt不会保留小数部分。number和bigInt不可以混用。bigInt不支持前置+号。

### 追问：如何判断数据类型

### 答：

使用`typeof`来判断数据类型，`typeof` 操作符会返回下列字符串之一：

-  "undefined"：表示值未定义；  
- "boolean"：表示值为布尔值；  
- "string"：表示值为字符串； 
- "number"：表示值为数值； 
- "object"：表示值为对象（而不是函数）或 null； 
- "function"：表示值为函数； 
- "symbol"：表示值为符号。
- "bigInt"：表示值为bigInt数值； 

>JavaScript所有值都设计成32位，其中最低的3位用来表述数据类型，object对应的值是000，null是用32个0表示。typeof就是根据最低的3位来判断数据类型，所以对null使用typeof会返回object

### 追问：typeof的不足和解决方案

### 答：

**缺点：**

- 无法区分object和null
- 不能区分对象的具体类型，Function，Array，Date等引用类型都返回object，无法具体区分。
- 对于NaN的判断，会返回number

**弥补：**

**`instanceof`** 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。返回的是一个布尔值。

```javascript
const arr = []
console.log(typeof arr);       //object
console.log(arr instanceof Array; //true)

```

`toString()` 方法也是弥补的手段，`toString()`函数用于将当前对象以字符串的形式返回，`toString()` 方法被每个 `Object` 对象继承。如果此方法在自定义对象中未被覆盖，`toString()` 返回 "[object *type*]"，其中 `type` 是对象的类型。但JavaScript的内置对象如Array，Boolean，String等，都重写了该 方法。

```JavaScript
let arr = [1,2,3];
let str = 'I am str';
let num = 18;
let obj = {}
console.log(arr.toString()); //1,2,3   
console.log(str.toString()); // I am str
console.log(num.toString()); // 18  
console.log(obj.toString()); // [object Object]
```

所以可以通过Object对象的toSting函数来判断对象类型。

```javascript
const toString = Object.prototype.toString;

toString.call(new Date); // [object Date]
toString.call(new String); // [object String]
toString.call(Math); // [object Math]

//Since JavaScript 1.8.5
toString.call(undefined); // [object Undefined]
toString.call(null); // [object Null]
```

### 追问：toString()和valueOf()的区别

### 答：

`valueOf()`和`toString()`都是属于Object的方法，不同的内置对象都重写了`valueOf()`方法。Object的`valueOf()` 方法返回指定对象的原始值。

​																				**不同类型对象的valueOf()返回的值**

| **对象** |                        **返回值**                        |
| -------- | :------------------------------------------------------: |
| Array    |                    返回数组对象本身。                    |
| Boolean  |                         布尔值。                         |
| Date     | 存储的时间是从 1970 年 1 月 1 日午夜开始计的毫秒数 UTC。 |
| Function |                        函数本身。                        |
| Number   |                         数字值。                         |
| Object   |                 对象本身。这是默认情况。                 |
| String   |                        字符串值。                        |
|          |          Math 和 Error 对象没有 valueOf 方法。           |

### 追问：instanceof的原理是什么

## 题2:JavaScript的数据类型转换

### 问： [] == ![]结果是什么？为什么？

### 答：

在转换操作数的类型时，`==`和`!=`操作符遵循如下规则：

- 当 `== `或者`!=`操作符两边的类型相同时，是不需要进行类型转换的。(废话)
- 当 `== `或者`!=`操作符两边的操作数不相等时，会先进行类型转换。

1. 如果任一操作数是布尔值，则将其转换为数值再比较是否相等。false 转换为 0，true 转换 为 1。 
2. 如果一个操作数是字符串，另一个操作数是数值，则尝试将字符串转换为数值，再比较是否 相等。 
3. 如果一个操作数是对象，另一个操作数不是，则先取得对象的原始值，再根据前面的规则进行比较。对象转换为原始值时要么使用其 toString() 方法，要么使用其 valueOf() 方法。JavaScript 内置的核心类先尝试使用 valueOf()，再尝试 toString()。但 Date 类是个例外，这个类执行 toString() 转换。

因为`!`的优先级大于`==`，所以会优先执行**![]**，

```javascript
!Boolean([])       //false
[] == false        //[] == ![]就等价于该表达式
//此时根据规则3,左边的表达式的转换如下
[].valueOf()  // []
[].toString() // ''
//此时在根据规则1
Number('') == Number(false)  //'' == false等价于该表达式
0 == 0        //最后结果为true
```

### 追问：为什么 0 == '0' 结果为 true , 0 == [] 结果为true, '0' == [] 却为 false?

### 答：

这个问题是`== 隐式转换规则`放弃了相等的传递性，是他的矛盾之处

```JavaScript
 0 == '0'
//采用规则2，转变后
0 == 0 
```

```JavaScript
0 == []
//采用规则3，转变后
0 == 0
```

```javascript
'0' == []
//对[]，使用valueOf()和toString()进行转换，变成‘’，此时表达式为
'0' == ''          //结果自然为false

```

### 追问：如何让if(a == 1 && a == 2)条件成立？

### 答：

利用对象会使用valueOf()和toString()获取原始值的方式

```javascript
var a = {
  value: 0,
  valueOf: function() {
    this.value++;
    return this.value;
  }
};
console.log(a == 1 && a == 2);//true
```

## 题3：谈谈你对闭包的理解

## 题4：谈谈你对原型链的理解

## 题5：谈谈你对argument的理解

### 问：arguments是什么？

`arguments`对象是所有（非箭头）函数中都可用的**局部变量**。你可以使用`arguments`对象在函数中引用函数的参数。此对象包含传递给函数的每个参数，第一个参数在索引0处

```javascript
function arg(){
	let name = arguments[0]
    let age = arguments[1]
    let addr = arguments[2]
	console.log(name,age,addr)  //gl 18 gz
}
arg('gl', 18, 'gz')
```

### 追问：arguments是数组吗？

### 答：

arguments不是一个数组，只不过它的属性从0开始排，依次为0，1，2...最后还有callee和length属性，但没有数组的方法。我们也把这样的对象称为类数组。常见的类数组有:

- 通过`getElementByTagName/ClassName/Name()`获得的`HTMLCollection`
- 通过`querySlector`获得的`nodeList`

### 追问：如何将类数组转变成数组

### 答：

```javascript
let analogousArray = {
	0: 'gul',
	1: 18,
	2: 'gz',
	length: 3
}
```

- Array.form()

  ```javascript
   let array = Array.from(analogousArray);
  ```

- 展开运算符

  ```javascript
  let array = [...arguments];
  ```

- concat+apply

  ```JavaScript
   let array = Array.prototype.concat.apply([], arguments);//apply方法会把第二个参数展开
  ```

- Array.prototype.slice.call()

  ```javascript
  let array = Array.prototype.slice.call(arguments);
  //slice是截取数组从start（包括该元素）到end（不包括该元素）的元素并返回。
  ```

## 题6：遍历(循环)语法

### 问：有哪些遍历的语法

- 基本for
- for...of
- for...in
- forEach
- map、reduce、filter、some、every等数组专用的方法

### 追问：forEach是否可以中断

### 答：

forEach无法使用break，return来中断循环。在回调函数内使用break并不会影响到外部的forEach。

```JavaScript
Array.prototype.MyForEach = function(fn){
    const length = this.length  		//遍历范围在第一次执行时就确定，不会更改
    for(let i = 0 ; i < length ; i++){
        fn(this[i],i,this);
    }
}
const arr = [1,2,3,4,5];
arr.MyForEach((item)=>{
    console.log(item);
    break;                     //添加break或者return
})

//已删除的项不会被遍历到。如果已访问的元素在迭代时被删除了（例如使用 shift，splice，pop）
//之后的元素将被跳过，但上诉代码并未实现
```

但我们可以通过抛出异常来中断循环

```javascript
const arr = [1,2,3,4,5];
try {
    arr.forEach(function(item) {
        if (item=== 2) throw item;
        console.log(item);
    });
} catch (e) {}
```

## 题目7：数组

### 问：判断数组中是否包含某个值

### 答

- indexOf：有返回相应下标，否返回-1
- includes：有返回true，否返回false
- findIndex：返回符合传入测试（函数）条件的数组元素，有符合就返回相应下标，反之返回-1
- find：返回符合传入测试（函数）条件的数组元素，有符合就返回相应元素，反之返回undefined
- 其他数组方式，filter，some，for循环等。

### 追问：数组扁平化的方式

### 答：

- flat()函数

  ```javascript
  //depth表示展开深度，默认为1，这里直接传入Infinity(无限大，所以不论多少层都可以展开)
  arr.flat([depth]) 
  ```

- 递归调用

- reduce

- 扩展运算符

- 正则加replace、toString加split，完全不推荐使用

### 追问：实现一个flat

### 答：

flat的特性总结

- 用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。
- 不传参数时，默认“拉平”一层，可以传入一个整数，表示想要“拉平”的层数。
- 传入 `<=0` 的整数将返回原数组，不“拉平”
- `Infinity` 关键字作为参数时，无论多少层嵌套，都会转为一维数组
- 如果原数组有空位，`Array.prototype.flat()` 会跳过空位。

flat的实现思路

- 遍历数组的每一个元素，可以采用forEach，for..of，map 等等
- 判断当前元素是否是数组，可以采用Array.isArray，instanceof，typeof等
- 将数组“拉平”一层，可以采用concat加扩展运算符，或者concat加apply

基础flat的实现

```javascript
//递归实现
const arr = [1, [2, [3, [4, 5]]], 6]
function flat(arr) {
    const newArr = [];
    arr.forEach((item) => {
        if(Array.isArray(item)) {
            const result = flat(item);
            newArr.push(...result); 
        } else {
            newArr.push(item)
        }
    })
    return newArr;
}
```

```JavaScript
//reduce实现
function flat(arr) {
    const newArr = arr.reduce((pre,cur)=>{
        return pre.concat(Array.isArray(cur) ? flat(cur) : cur);
    },[])
    return newArr;
}
```

可控制层数的flat实现，就是在原先reduce实现的基础上，增加一个num变量，每拉平一层，num的值就减一，当num为0的时候，就停止拉平

```JavaScript
const arr = [1, [2, [3, [4, 5]]], 6]
function flat(arr, num = 1) {
    let newArr = []
    if (num > 0) {
        newArr = arr.reduce((pre, cur) => {
            return pre.concat(Array.isArray(cur) ? flat(cur, num - 1) : cur)
        }, [])
    } else {
        newArr = arr.slice();
    }
    return newArr;
}
```

## 题7：高阶函数

### 问：什么是高阶函数

### 答：

`一个函数`就可以接收另一个函数作为参数或者返回值为一个函数，`这种函数`就称之为高阶函数。

### 追问：数组中的高级函数有哪些

### 答：

map 、reduce 、filter 、some 、every、 find 、findIndex、sort

### 追问：实现一个map

### 答：

```JavaScript
Array.prototype.map = function(callbackFn, thisArg) {
  // 处理数组类型异常
  if (this === null || this === undefined) {
    throw new TypeError("Cannot read property 'map' of null or undefined");
  }
  // 处理回调类型异常
  if (Object.prototype.toString.call(callbackfn) != "[object Function]") {
    throw new TypeError(callbackfn + ' is not a function')
  }
  // 草案中提到要先转换为对象
  let O = Object(this);
  let T = thisArg;


  let len = O.length >>> 0;
  let A = new Array(len);
  for(let k = 0; k < len; k++) {
    // 还记得原型链那一节提到的 in 吗？in 表示在原型链查找
    // 如果用 hasOwnProperty 是有问题的，它只能找私有属性
    if (k in O) {
      let kValue = O[k];
      // 依次传入this, 当前项，当前索引，整个数组
      let mappedValue = callbackfn.call(T, KValue, k, O);
      A[k] = mappedValue;
    }
  }
  return A;
}
```

 length >>> 0, 字面意思是指"右移 0 位"，但实际上是把前面的空位用0填充，这里的作用是保证len为数字且为整数。

```JavaScript
null >>> 0  //0

undefined >>> 0  //0

void(0) >>> 0  //0

function a (){};  a >>> 0  //0

[] >>> 0  //0

var a = {}; a >>> 0  //0

123123 >>> 0  //123123

45.2 >>> 0  //45

0 >>> 0  //0

-0 >>> 0  //0

-1 >>> 0  //4294967295

-1212 >>> 0  //4294966084
```

更多访问https://zhuanlan.zhihu.com/p/90017386





