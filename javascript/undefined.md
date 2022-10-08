1JavaScript诞生于1995年，主要包含ECMAScript，DOM，BOM。其中ECMA-262做为ECMAScript的标准规范，定义了该语言的语法，类型，语句，关键字，保留字，操作符以及对象。

ECMA-262的第一版本质上与Netscape的JavaScript1.1相同，只不过删除了所有针对浏览器的代码并作了一些较小的改动。ECAM-262要求支持Unicode标准，而且对象也变成了与平台无关的。

ECMA-262第二版主要是编辑加工的结果，是为了与ISO/IEC-16262保持严格一致，没有任何新增，修改或者删除处理。

ECMA-262第三版是第一次真正的修改，涉及字符串处理，错误定义和数值输出，这一版本还新增了对正则表达式，新控制语句try catch异常的处理，这标志着ECMAScript成为了一门真正的编程语言。

ECMA-262第四版对这门语言进行了一次全面的修订，出台的标准几乎在第三版的基础上完全定义了一门新语言。第四版不仅包含了强类型变量，新语句和新数据结构，真正的类和经典继承，还定义了与数据交互的新方式。由于第四版跨度太大，TC39下属的一个小组提出了一个名为ECMAScript3.1的替代性建议，只对这门语言进行了较少的改进。第四版在发布前被放弃，也就是说没有第四版发布。ECMAScript3.1过渡为第五版。

ECMA-262第五版于2009年12月3日正式发布，第五版力求厘清第三版中已知的歧义并添加了新的功能。新功能包括原生JSON对象，继承的方法和高级属性定义，另外还包含一种严格模式，对ECMAScript引擎解释和执行代码进行了补充说明。第五版在2011年6月发布了一个维护性修订版，更正了规范中的错误。

ECMA-262第六版俗称ES6或ES2015，于2015年6月发布，正式支持了类、模块、迭代器、生成器、箭头函数、期约、反射、代理和众多新的数据类型。

ECMA-262第七版，于2016年发布，只包含少量语法层面的增强，如Array.prototype.includes和指数操作符。

ECMA-262第八版，完成于2017年6月，增加了异步函数，SharedArrayBuffer，Object.values、Object.entries、Object.getOwnPropertyDescriptors。

ECMA-262第九版，发布于2018年6月，支持异步迭代，剩余属性，Promise.finally等。

ECMA-262第十版，发布于2019年6月，增加了Array.prototype.flat/flatMap、String.prototype.trimStart/trimEnd、Object.fromEntries、Symbol.prototype.description。明确了Function.prototype.toString返回值，固定了Array.prototype.sort顺序以及catch可先绑定。

## DOM 文档对象模型

1998年10月，DOM Level 1称为W3C的推荐标准，由DOM Core 和 DOM HTML组成。DOM Level 1的目标是映射文档结构。

DOM Level 2增加了对鼠标和用户界面的事件、范围、遍历的持之，并且支持了CSS。

DOM Level 3增加了统一加载和保存文档的方法，验证文档的模块。

目前W3C不再按照Level来维护DOM了，改为DOM Living Standard来维护，称为DOM4，新增的内容包括替代Mutation Events的Mutation Observers。

DOM Level 0并不存在，只是一个参照点，可以看做是IE4和网景4最初支持的DHTML。

## BOM 浏览器对象模型

用于支持访问和操作浏览器的接口，在H5以前他是没有相关标准的JavaScript实现，H5的出现统一了BOM接口的实现标准。

BOM 主要针对浏览器窗口和子窗口，比如弹出新窗口，移动，缩放和关闭窗口，navigator对象，location对象，screen对象，performance对象，对cookie的支持，还有XMLHttpRequest。

## 小结

JavaScript是一门用来与网页交互的脚本语言，包含ECMAScript、DOM、BOM。

ECMAScript由ECMA-262定义并提供核心功能。文档对象模型提供与网页内容交互的方法和接口。浏览器对象模型提供了浏览器交互的方法和接口。

JavaScript的这三个部分得到了五大浏览器(IE、Firefox、Chrome、Safari和Opera)不同程度的支持。所有浏览器基本对ES5提供了完善的支持，而对ES6和ES7的支持度也在不断提升。这些浏览器对DOM的支持各不相同，但对Level 3的支持日趋规范。HTML5中收录的BOM会因浏览器而异，不过开发者仍然可以假定存在很大的一部分公共特性。