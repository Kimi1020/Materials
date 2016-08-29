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