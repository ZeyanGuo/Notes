# lodash.isArrayLike
isArrayLike用于判断传入的对象是否是类数组，那么什么是类数组呢？isArrayLike是通过什么方式来进行判断的呢？以下将进行分析：

## 数组，对象，类数组
数组定义：单独变量存储的一系列的值，其索引从0开始自然增长，其中值可以是任何的js数据，并且包含一个名为Length的属性，用于表示数组元素的个数。

对象定义：单独变量可以包含多个其它变量，其它变量是一组无序的由键-值对组成的数据集。

类数组：只要包含从零开始，且自然递增的整数作为健名，并且定义了length表示元素个数的对象。

数组-对象-类数组代码：
```
var arr = [1,2,3]; //数组
var obj = {a:1,b:2,c:3}; //对象
var isArrLike = {0:1,1:2,2:3,length:3}; //类数组
```

## lodash.isArrayLike方法
要清楚isArrayLike方法是如何判断类数组对象，那么接下来首先分析源代码：

```
/***** isArrayLike.js *****/ 
import isLength from './isLength.js';

function isArrayLike(value){
    return value != null && typeof value != 'function' && isLength(value.length);
}

export default isArrayLike;
/***** isArrayLike.js *****/

/***** isLength *****/
const MAX_SAFE_INTEGER = 9007199254740991;

function isLength(value){
    return typeof value == 'number' && value > -1 && value % 1 == 0 && value <= MAX_SAFE_INTEGER;
}

export default isLength;
/***** isLength *****/
```

从代码中可以看出，首先isArrayLike对是否是null进行了判断，然后用```typeof value != 'function'```进行了一次判断，最后在判断是否存在length属性且length属性是否是正整数。

初次看到上述验证时，对于isLength和非null判断并没有疑惑，但是对于```typeof != 'function'``` 却存在疑问：

    1. 这个typeof有何用处？
    2. 为何要做typeof != 'function'判断?

经过查阅资料，得到以下信息：
#### typeof 
用于检测js变量的类型，其返回类型为字符串，值包括如下几种：
1. 'undefined'  -- 未定义的变量或值
2. 'boolean'    -- 布尔类型的变量或值
3. 'string'     -- 字符串类型的变量或值
4. 'number'     -- 数字类型的变量或值
5. 'object'     -- 对象类型的变量或值，或者null
6. 'function'   -- 函数类型的变量或值

#### function函数的length属性
对于function函数来说其length属性返回的是函数的形参个数

由上述资料可得知，在isArrayLike方法的collection传入为函数时，由于Function.length属性返回的是形参个数，因此如果不加```typeof != 'function'```会导致函数也能通过判断并且返回是类函数，那么该判断是必要的。


以上内容如有错误还望斧正。