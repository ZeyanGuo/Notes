# lodash.reduce
lodash文档解释：通过 iteratee 遍历集合中的每个元素。 每次返回的值会作为下一次 iteratee 使用。 如果没有提供 accumulator，则集合中的第一个元素作为 accumulator。 iteratee 会传入4个参数：(accumulator, value, index|key, collection)。

当看到官方文档解释的时候有点懵，接下来我们通过源代码逐一分析。

lodash.reduce相关的源代码：
```
/***** lodash.reduce *****/
import arrayReduce from './.internal/arrayReduce.js'
import baseEach from './.internal/baseEach.js'
import baseReduce from './.internal/baseReduce.js'

function reduce(collection, iteratee, accumulator) {//此处可见reduce支持3个参数(collection-可遍历对象，iteratee-处理函数，accumulator-初始值)
  const func = Array.isArray(collection) ? arrayReduce : baseReduce //可遍历对象是否是数组，如果是则调用arrayReduce，否则调用baseReduce
  const initAccum = arguments.length < 3//判断是否传入accumulator，如果未传入则使用可遍历对象的第一个值作为初始值
  return func(collection, iteratee, accumulator, initAccum, baseEach)
}

export default reduce
/***** lodash.reduce *****/

/***** lodash.arrayReduce *****/
function arrayReduce(array, iteratee, accumulator, initAccum) {//直接遍历数组，如果accumulator有值就使用其作为初始值，否则使用数组的第一个值作为初始值
  let index = -1
  const length = array == null ? 0 : array.length

  if (initAccum && length) {
    accumulator = array[++index]
  }
  while (++index < length) {
    accumulator = iteratee(accumulator, array[index], index, array)//对数组中的内容按照iteratee方法进行叠加
  }
  return accumulator
}

export default arrayReduce

/***** lodash.arrayReduce *****/

/***** lodash.baseReduce *****/
function baseReduce(collection, iteratee, accumulator, initAccum, eachFunc) {
  eachFunc(collection, (value, index, collection) => {
    accumulator = initAccum
      ? (initAccum = false, value)//此处的括号不理解，请阅读后续文章
      : iteratee(accumulator, value, index, collection)
  })
  return accumulator
}

export default baseReduce
/***** lodash.baseReduce *****/

/***** lodash.baseEach *****/
import baseForOwn from './baseForOwn.js'
import isArrayLike from '../isArrayLike.js'

function baseEach(collection, iteratee) {
  if (collection == null) {
    return collection
  }
  if (!isArrayLike(collection)) {//isArrayLike用于判断是否是类数组，具体判断方法可参考文章(https://note.youdao.com/)
    return baseForOwn(collection, iteratee)//通过Object.keys方法来遍历非类数组对象。
  }
  const length = collection.length
  const iterable = Object(collection)
  let index = -1

  while (++index < length) {
    if (iteratee(iterable[index], index, iterable) === false) {
      break
    }
  }
  return collection
}

export default baseEach
/***** lodash.baseEach *****/
```

上述类容如果和我一样对JS用法理解不透彻，那么可能就回遇到以下的问题：
    
    1. (varible_01 = value_01, varible_02 = value_02, ... , value_last)在js中是如何解析的？
    2. isArrayLike中所指的类数组究竟是什么？
    3. Object.keys是什么？怎么用？

接下来就对上述问题逐一解释，希望能给大家帮助。

## 问题解答(一) - JS中括号的使用

js中括号有以下的使用场景

 1. 提高优先级
 2. 函数参数用括号包含
 3. JS中立即函数的使用
 4. 条件中判断语句包含于括号
 5. **执行单个或多个表达式** <- 

lodash.reduce代码中遇到的 ```(initAccum = false, value)```就是括号用法5的用例，括号的第五种用法是首先执行所有表达式，然后将最后一个表达式值返回。

例子：
```
console.log((1, 2, variable_01 = 3, 3 + 4, variable_02 = 4));// console: 4
console.log(variable_01);// console: 3
console.log(variable_02);// console: 4
```

## 问题解答(二) - 对象，类数组，数组

数组定义：单独变量存储的一系列的值，其索引从0开始自然增长，其中值可以是任何的js数据，并且包含一个名为Length的属性，用于表示数组元素的个数。

对象定义：单独变量可以包含多个其它变量，其它变量是一组无序的由键-值对组成的数据集。

类数组：只要包含从零开始，且自然递增的整数作为健名，并且定义了length表示元素个数的对象。

数组-对象-类数组代码：
```
var arr = [1,2,3]; //数组
var obj = {a:1,b:2,c:3}; //对象
var isArrLike = {0:1,1:2,2:3,length:3}; //类数组
```
如果想进一步了解isArrayLike方法的细节，请点击[lodash.isArrayLike详解-传送门]()

## 问题解答(三) - Object.keys
由于是基础JS方法介绍，此处大量引用MDN文章 [Object.keys() - MDN传送门]()毕竟没有比官方解释更权威的答案了:)

Object.keys() 方法会返回一个由一个给定对象的**自身**可枚举属性组成的数组，数组中属性名的排列顺序和使用 for...in 循环遍历该对象时返回的顺序一致 。如果对象的键-值都不可枚举，那么将返回由键组成的数组。

**描述**：Object.keys 返回一个所有元素为字符串的数组，其元素来自于从给定的object上面可直接枚举的属性。这些属性的顺序与手动遍历该对象属性时的一致。

例子：
```
// simple array
var arr = ['a', 'b', 'c'];
console.log(Object.keys(arr)); // console: ['0', '1', '2']

// array like object
var obj = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.keys(obj)); // console: ['0', '1', '2']

// array like object with random key ordering
var anObj = { 100: 'a', 2: 'b', 7: 'c' };
console.log(Object.keys(anObj)); // console: ['2', '7', '100']

// getFoo is a property which isn't enumerable
var myObj = Object.create({}, {
  getFoo: {
    value: function () { return this.foo; }
  } 
});
myObj.foo = 1;
console.log(Object.keys(myObj)); // console: ['foo']
```

还有一个与Object.keys相似的方法Object.entries，用于返回一个给定对象**自身**可枚举属性的键值对***数组***，具体的用法可参考[Object.entries() - MDN传送门](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/entries)

以上内容如有错误还望斧正。