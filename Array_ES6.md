#ES6 learning

###数组的扩展
####Array.from()
[Reference：  Array.from ()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
Array.from` 方法用于将两类对象转为真正的数组： 类似数组的对象（array-Like objec）和可遍历（iterable）的对象（包括ES6的Set 和 Map）

```Javascript
let arrayLike ={
   '0' : 'a',
   '1' : 'b',
   '2' : 'c',
   length: 3
};

//ES5的写法
var arr = [].slice.call(arrayLike); //[a,b,c]

//ES6的写法
var arr = Array.from(arrayLike); //[a,b,c]
```

实际应用中，常见的类似数组的对象都是DOM操作返回的NodeList集合，以及函数内部的arguments对象。Array.from都可以将它们转为真正的数组。

**所谓类似数组的对象，本质特征只有一个，即必须有length属性。**

```JavaScript
let ps = document.querySelectorAll('p');
Array.from(ps).forEach(function (p){


});


function foo() {
  var args = Array.from(arguments);
  //...
}
```

```javaScript
Array.from('hello'); //[h,e,l,l,o]

let nameSet = new Set(['a', 'b']);
Array.from(nameSet); //['a', 'b']


//如果是一个真正的数组，则会返回一个一模一样的新数组。

Array.from([1,2,3]);//[1,2,3]
```

Array.from 还接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理， 将处理后的值放入返回的数组。
```Javascript
 Array.from(arrayLike, x=>x*x);
//等同于
 Array.from(arrayLike).map(x=>x*x);
 
```

如果map函数里面用到了this关键字，还可以传入Array.from的第三个参数， 用来绑定this

值得提醒的是， 扩展运算符（...）也可以将某些数据结构转为数组

```Javascript
function foo(){
   var args = [...arguments];
}

[...document.querySelectorAll('div')]
```


####Array.of()
Array.of方法用于将一组值，转换为数组，这个方法的主要目的，是弥补数组构造函数Array()的不足。因为参数个数的不同，会导致Array（）的行为有差异。

```Javascript
Array() // []
Array(3) // [,,,]
Array(1,2,3) //[1,2,3]


Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1

```

####Array.prototype.copyWithin()
数组实例的copyWithin方法，在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。也就是说，使用这个方法，会修改当前数组。

```Javascript
Array.prototype.copyWithin(target, start=0, end=this.length)

它接受三个参数。
-target（必需）： 从该位置开始替换数据。
-start （可选）： 从该位置开始读取数据，默认为0.如果为负值，表示倒数。
-end (可选) ： 到该位置钱停止读取数据，默认等于数组长度。如果为负值，表示倒数。


[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]
// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 将3号位复制到0号位
[].copyWithin.call({length:3, 3:1}, 0, 3);
// {0: 1, 3: 1, length: 5}

```


####Array.prototype.find && Array.prototype.findIndex
数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数时一个回调函数，所有数组成员一次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。

```Javascript
[1,2,3,4].find(n =>n>1)//[2,3,4]

find 方法的回调函数可以接受三个参数，依次为value， index， array， 当前的值，当前的位置和原数组。

数组实例的findIndex方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。

[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2

**这两个方法都可以接受第二个参数，用来绑定回调函数的this对象。**


这两个方法都可以发现NaN，弥补了数组的IndexOf方法的不足。

[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

####Array.prototype.fill() 
fill 方法使用给定值，填充一个数组。
fill方案还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。

```Javascript
['a','b','c'].fill(7,1,2); //['a', 7, 'c']


```

----------------------------------------------

ES6提供了三个新的方法，用于遍历数组。

####Array.prototype.entries()

####Array.prototype.keys()

####Array.prototype.values()

```Javascript
for（let index of ['a','b'].keys()){
   consoloe.log(index);
}
// 0
// 1

for(let elem of ['a', 'b'].values()){
   console.log(elem);
}
// 'a'
// 'b'

for(let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
```
// 0 "a"
// 1 "b"

如果不使用for...of循环，可以手动调用遍历器对象的next方法，进行遍历。
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']

####Array.prototype.includes()
方法返回一个布尔值，表示某个数组是否包含给定的值。与字符串的includes方法类似。该方法属于ES7，但Babel转码器已经支持。

```Javascript
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, NaN].includes(NaN); // true

该方法的第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。

[NaN].indexOf(NaN)
// -1

[NaN].includes(NaN)
// true

下面代码用来检查当前环境是否支持该方法，如果不支持，部署一个简易的替代版本。

const contains = (()=>
  Array.prototype.includes
  ? (arr, elem) => arr.includes(elem)
  : (arr, value) => arr.some(el =>el ===value)
  )();

```

####数组的空位
new Array(3)//[,,,]
ES5对空位的处理，已经很不一致了,大多数情况下会忽略空位
forEach(), filter(), every() 和some()都会跳过空位。
map()会跳过空位，但会保留这个值
join()和toString()会将空位视为undefined，而undefined和null会被处理成空字符串。


```Javascript
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"

```

**ES6则是明确将空位转为undefined。**

```Javascript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]

[...['a',,'b']]
// [ "a", undefined, "b" ]

//copyWithin()会连空位一起拷贝。
[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]

//entries()、keys()、values()、find()和findIndex()会将空位处理成undefined。
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```
