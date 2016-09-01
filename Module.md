#Module
1. 严格模式
2. export命令
3. import命令
4. 模块的整体加载
5. export default命令
6. 模块的继承
7. ES6模块加载的实质
8. 循环加载
9. 跨模块常量
10. ES6模块的转码

历史上，Javascript一直没有Module体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。在ES6之前，社区制定了一些模块加载方案，最主要的是CommonJS和AMD两种。 ES6在语言规格的层面上，实现了模块功能，而且实现得相当简单，完全可以取代现有的CommonJS和AMD规范，成为浏览器和服务器通用的模块解决方案。

ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS和AMD模块，都只能在运行时确定这些东西。比如，CommonJS模块就是对象，输入时必须查找对象属性。

```Javascript
// CommonJS module
let { stat, exists, readFile } = require('fs');

// equal to
let _fs = require('fs');
let stat = _fs.stat, exists= _fs.exists, readFile = _fs.readfile;

```
上面代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取3个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”

ES6模块不是对象，而是通过export命令显式指定输出的代码，输入时也采用静态命令的形式。
```Javascript
import {stat, exists, readFile} from 'fs'
```
上面代码的实质是从fs模块加载3个方法，其他方法不加载。这种加载称为“编译时加载”

###严格模式
严格模式主要有以下限制
	1.变量必须声明后再使用
	2.函数的参数不能有同名属性，否则报错
	3.不能使用with语句
	4.不能对只读属性赋值，否则报错
	5.不能使用前缀0表示八进制数，否则报错
	6.不能删除不可删除的属性
	7.不能删除变量 `delete prop`, 只能删除delete global[prop]
	8.eval不能再它的外层作用域引入变量
	9.eval和arguments不能重新赋值
	10.arguments不能自动反应函数参数的变化
	11.不能使用arguments.caller 和arguments.callee
	12.禁止this指向全局对象
	13.不能使用fn.caller 和fn.arguments获取函数调用的堆栈
	14.增加了保留字（比如protected， static 和interface）



###export命令
模块功能主要由两个命令构成：export和import。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用export关键字输出该变量。下面是一个JS文件，里面使用export命令输出变量

```Javascript
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;

// equal to
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;
export {firstName, lastName, year};

```
export也可以输出函数或类
```Javascript
export function multiply(x, y) {
	return x * y;
}

// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass';
let o = new MyClass();

//通常情况下，export输出的变量就是本来的名字，但是可以使用as关键字重命名
function v1() { ... }
function v2() { ... }

export {
	v1 as streamV1,
	v2 as streamV2,
	v2 as streamLatestVersion
};

//需要特别注意的是，export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。 它们的实质是，在接口名与模块内部变量之间，建立了一一对应的关系。
// 报错
export 1;

// 报错
var m = 1;
export m;


// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};


同样的，function和class的输出，也必须遵守这样的写法。
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};


//另外，export语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。 下面代码输出变量foo，值为bar，500毫秒之后变成baz
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);


```
最后，export命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错，下一节的import命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了ES6模块的设计初衷。

###import命令
使用export命令定义了模块的对外接口以后，其他JS文件就可以通过import命令加载这个模块（文件）。

```Javascript
import {firstName, lastName, year} from './profile';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
//上面代码的import命令，就用于加载profile.js文件，并从中输入变量。import命令接受一个对象（用大括号表示），里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（profile.js）对外接口的名称相同。

```
**注意，import命令具有提升效果，会提升到整个模块的头部，首先执行。**

如果在一个模块之中，先输入后输出同一个模块，import语句可以与export语句写在一起。
```Javascript
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;
```
上面代码中，export和import语句可以结合在一起，写成一行。但是从可读性考虑，不建议采用这种写法，而应该采用标准写法。


###模块的整体加载
除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面
```Javascript
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

import * as circle from './circle';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```

###export default命令
为模块指定默认输出
```Javascript
// export-default.js
export default function () {
  console.log('foo');
}

import customName from './export-default';
customName();
```
export default命令用在非匿名函数前，也是可以的。
```Javascript
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;
```
上面代码中，foo函数的函数名foo，在模块外部是无效的。加载的时候，视同匿名函数加载。

```Javascript
// 输出
export default function crc32() {
  // ...
}
// 输入
import crc32 from 'crc32';

// 输出
export function crc32() {
  // ...
};
// 输入
import {crc32} from 'crc32';


```
上面代码的两组写法，第一组是使用export default时，对应的import语句不需要使用大括号；第二组是不使用export default时，对应的import语句需要使用大括号。

export default命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此export deault命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能对应一个方法。

本质上，export default就是输出一个叫做default的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。
```Javascript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as xxx } from 'modules';
// 等同于
// import xxx from 'modules';
```
正是因为export default命令其实只是输出一个叫做default的变量，所以它后面不能跟变量声明语句。
```Javascript
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;
```

###模块的继承
模块之间也可以继承。

假设有一个circleplus模块，继承了circle模块。

```Javascript
//circleplus.js
/*
上面代码中的export *，表示再输出circle模块的所有属性和方法。
注意，export *命令会忽略circle模块的default方法。
然后，上面代码又输出了自定义的e变量和默认方法。
*/
export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
	return Math.exp(e);
};
//也可以将circle的属性或方法，改名后再输出
export {area as circleArea} from 'circle'

// main.js
import * as math from 'circleplus';
import exp from 'circleplus';
console.log(exp(math.e));

```

### ES6模块加载的实质
ES6模块加载的机制，与CommonJS模块完全不同。CommonJS模块输出的是一个值的拷贝，而ES6模块输出的是值的引用。

```Javascript
/* 
 CommonJS模块输出的是被输出值的拷贝，也就是说，一旦输出一个值，
 模块内部的变化就影响不到这个值。 

*/
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};

// main.js
var mod = require('./lib');

console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3

```

上面代码说明，lib.js模块加载以后，它的内部变化就影响不到输出的mod.counter了。这是因为mod.counter是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。
```Javascript

// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};


$ node main.js
3
4
```
ES6模块的运行机制与CommonJS不一样，它遇到模块加载命令import时，不会去执行模块，而是只生成一个动态的只读引用。等到真的需要用到时，再到模块里面去取值，换句话说，ES6的输入有点像Unix系统的“符号连接”，原始值变了，import输入的值也会跟着变。因此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。


```Javascript
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4


//再举一个出现在export一节中的例子
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);

// m2.js
import {foo} from './m1.js';
console.log(foo);
setTimeout(() => console.log(foo), 500);


//上面代码中，m1.js的变量foo，在刚加载时等于bar，过了500毫秒，又变为等于baz
$ babel-node m2.js

bar
baz
```
上面代码表明，ES6模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。

由于ES6输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。
```Javascript
// lib.js
export let obj = {};

// main.js
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```
上面代码中，main.js从lib.js输入变量obj，可以对obj添加属性，但是重新赋值就会报错。因为变量obj指向的地址是只读的，不能重新赋值，这就好比main.js创造了一个名为obj的const变量。


最后，export通过接口，输出的是同一个值。不同的脚本加载这个接口，得到的都是同样的实例
```Javascript
// mod.js
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}

export let c = new C();

// x.js
import {c} from './mod';
c.add();

// y.js
import {c} from './mod';
c.show();

// main.js
import './x';
import './y';


$ babel-node main.js
1
```

###循环加载
“循环加载”（circular dependency）指的是，a脚本的执行依赖b脚本，而b脚本的执行又依赖a脚本。

#####CommonJS模块的循环加载

CommonJS模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。

CommonJS模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

```Javascript
//a.js
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');

//b.js
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');

/*
上面代码之中，b.js执行到第二行，就会去加载a.js，这时，就发生了“循环加载”。系统会去a.js模块对应对象的exports属性取值，可是因为a.js还没有执行完，从exports属性只能取回已经执行的部分，而不是最后的值。

a.js已经执行的部分，只有一行。
exports.done = false;

因此，对于b.js来说，它从a.js只输入一个变量done，值为false。

然后，b.js接着往下执行，等到全部执行完毕，再把执行权交还给a.js。于是，a.js接着往下执行，直到执行完毕。我们写一个脚本main.js，验证这个过程。
*/

var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);

$ node main.js

在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true

```
上面的代码证明了两件事。一是，在b.js之中，a.js没有执行完毕，只执行了第一行。二是，main.js执行到第二行时，不会再次执行b.js，而是输出缓存的b.js的执行结果，即它的第四行。总之，CommonJS输入的是被输出值的拷贝，不是引用。 另外，由于CommonJS模块遇到循环加载时，返回的是当前已经执行的部分的值，而不是代码全部执行后的值，两者可能会有差异。所以，输入变量的时候，必须非常小心。

#####ES6模块的循环加载
ES6处理“循环加载”与CommonJS有本质的不同。ES6模块是动态引用，如果使用import从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

```Javascript
// a.js如下
import {bar} from './b.js';
console.log('a.js');
console.log(bar);
export let foo = 'foo';

// b.js
import {foo} from './a.js';
console.log('b.js');
console.log(foo);
export let bar = 'bar';


$ babel-node a.js
b.js
undefined
a.js
bar
```
上面代码中，由于a.js的第一行是加载b.js，所以先执行的是b.js。而b.js的第一行又是加载a.js，这时由于a.js已经开始执行了，所以不会重复执行，而是继续往下执行b.js，所以第一行输出的是b.js。

接着，b.js要打印变量foo，这时a.js还没执行完，取不到foo的值，导致打印出来是undefined。b.js执行完，开始执行a.js，这时就一切正常了。

###跨模块常量
上面说过，const声明的常量只在当前代码块有效。如果想设置跨模块的常量（即跨多个文件），可以采用下面的写法。
```Javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```










































