## 当心隐式的强制转换

### 重点
+ **类型错误可能被强制的隐式转换所隐藏**
+ **重载的运算符+要进行加法运算还是字符串连接操作取决于其参数类型**
+ **对象通过valueOf强制转换为数字，通过toString强制转换为字符串**
+ **具有valueOf方法的对象应该实现toString方法，返回一个valueOf方法产生的数字的字符串表示**
+ **测试一个值是否为未定义的值，应该使用typeof或者与undefinded进行比较而不是使用真值运算**

算术运算符两边有字符和数字的情况：
```
 2 + 3; //5
 'hello' + ' world';//'hello world'
 1 + 2 + '3'; //'33'
 1 + '2' + 3; //'123'
 '17' * 3; //51
 '8' | '1'; //9
 '12' - '3';//9
 '12' / '3';//4
 '12' % '3';//0
 '12' % '5';//2
```
很明显当为加运算符时，`JavaScript`更偏向于字符串的拼接；而其他情况都会转换为数字操作。

`JavaScript`在计算时的强制类型转换会隐藏类型错误：
```
3 + true;//4
3 + null;//null
3 + undefined;//NaN
```
从上面我们可以看出，结果为`true`、`null`和`undefined`的变量在算术运算中不会导致失败，而是相对应的转换为`1`、`0`和`NaN`这些值。这其中特殊的浮点数值`NaN`，代表的是`not a number`的意思。
当遇到不能强制转换成数字的变量时就会被转成`NaN`！并且`NaN`有一个特点——不等于其本身，即：`NaN === NaN;//false`。我们可以通过标准的库函数`isNaN`来判断是否为`NaN`：
```
isNaN(NaN);//true
isNaN(undefined);//true
isNaN('str');//true
isNaN({});//true
isNaN({valueOf: 'str'});//true
```
从上面我们可以看到，`isNaN`判断出了`NaN`，但是由于有强制类型转换的缘故，当其他无法转换为数字的变量都会判断为`NaN`！
因为`NaN`是`JavaScript`唯一不等于其本身的值，所以通过以下函数能判断出是否真为`NaN`：
```
function isReallyNan(x) {
  return x !== x;
}
```

我们在调试代码时，有时想要打印一下变量的内容：
```
'the Math object: ' + Math;//"the Math object: [object Math]"
'the JSON object: ' + JSON;//"the JSON object: [object JSON]"
```
上面的`Math`和`JSON`对象通过调用自身`toString`方法将对象转为字符串，类似的对象也可以通过其`valueOf`方法转为数字：
```
'J' + {toString(){return 'S';}};//'JS'
'2' +  {valueOf(){return 3;}};//'23'
```
通过上面我们发现，当我们复写`toString`和`valueOf`其中的某一个的时候，则在进行强制转换为原始值时，会优先调用被复写的那个方法。
当两个都被复写时：
```
var obj = {
  toString(){return '[object MyObject]'},
  valueOf(){return 17}
};
'object:' + obj;//"object:17"
```
这个例子说明，`valueOf`方法才是真正为那些代表数值的对象(如Number对象)而设计的。对于这些对象，`toString`和`valueOf`方法应该返回一致的结果（相同的数字字符串活数值表示），因此不管是对象的链接还是相加，重载运算符`+`总是一致的行为。一般情况下，字符串强制转换比数字的强制转换更常见、更有用，最好避免使用`valueOf`方法，除非对象的确是一个数字的抽象，并且`obj.toString()`能产生一个`obj.valueOf()`的字符串表示。

`JavaScript`有7个假值： false、0、-0、’‘、NaN、null和undefined。
检查参数是否为`undefined`，用`typeof`方式更为严格。
