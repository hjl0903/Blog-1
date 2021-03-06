# 编码规则

time: 2019.7.8  
author: heyunjiang

目录

[1. 基础知识](#1-基础知识)  
[2. 编码类型](#2-编码类型)  
&nbsp;&nbsp;[2.1 ASCII编码](#2.1-ASCII编码)  
&nbsp;&nbsp;[2.2 Unicode编码](#2.2-Unicode编码)  
&nbsp;&nbsp;[2.3 utf-8编码](#2.3-utf-8编码)  
&nbsp;&nbsp;[2.4 utf-16编码](#2.4-utf-16编码)  
&nbsp;&nbsp;[2.5 base64编码](#2.5-base64编码)  
[3. javascript 中的编码](#3-javascript-中的编码)  
[4. 中文编码](#4-中文编码)  
[5. 数组及字符串的 length](#5-数组及字符串的-length)  

> 这里为什么要总结这篇文章，因为编码在平时用的很少，但是也算是弥补了自己内心对js编码这一块的空缺，让自己以后面对编码不再恐惧，自己也明白了js采用unicode字符集和utf-16编码

首先，我们先明确什么是 `字符集`，什么又是 `编码方案`，它们之间有什么关系?  
我们日常开发写的字符串、保存在磁盘上的数据、运行在浏览器中的效果，他们之间的数据是如何转换的呢？

**关键词**：字节、fromCodePoint、codePointAt、encodeURI、encodeURIComponent

## 1 基础知识

1. 字符：就是编码写的，比如 'a, b, 中文，*, 2 ...' 等
2. 字符集：字符的集合，ASCII字符集、unicode字符集、GB2312字符集。javascript 采用 `unicode` 字符集，unicode 编码采用的都是 unicode 字符集，也就是 utf-8、utf-16 都是采用 unicode
3. 字符编码方案：是将字符处理为指定集合中的对象(汇编？bit、自然数队列、8位组或者电脉冲)，方便计算机存储、数据传递。`utf-8`、`utf-16`、`ASCII`
4. 代码编写：编写代码我们采用 `utf-8`，少有采用 `gb-2312 | GBK` 的，存储在计算机中还是 `0|1`
5. 但是存储在内存中的数据，javascript 采用 `UTF-16` 编码，也就是 `UCS-2`编码方式，每一个编码单元由一个 16 位的二进制数表示，每个字符由1个或2个单元表示(一般一个)

### 位、字节、字关系

位：`bit` , 计算机存储数据的最小单位，每一位状态只能是0或1

字节：`Byte` , `1Byte = 8bit` ，2^8-1 = 255, 计算机存储的基本计量单位(与bit最小存储单位不同)。1个字节存储1个英文字母和半个汉字，即 `1Byte = 1字母`，`2Byte = 1个汉字`

字(字符)：8位机，字长8位，`1个字 = 8位 = 1Byte`；16位机，字长16位，`1个字 = 16位 = 2Byte`。计算机数据处理和运算的单位

## 2 编码类型

### 2.1 ASCII编码

编码范围：大写字母、小写字母、一些符号，仅支持英文，不支持汉字等其他语言

编码长度：一个字节

ASCII字符集：字母、数字、符号

ASCII编码：将ASCII字符编码，最后用 7bit 或者 8bit 来表示这个字符，所以最多只能表示 2^8 个字符，也就是256

### 2.2 Unicode编码

编码范围：所有语言，包括汉字、英文、韩文等

编码长度：2个字节(如果非常偏僻的字符，4个字节)，现在长度已经不止2字节了

概念：作为一种编码方法，为每种语言中的每个字符设定了统一并且唯一的二进制编码。

存储：所有的文字都采用2个字节存储，二进制，英文编码高位字节填0

**表示**：`\u` + 一组十六进制。这里十六进制长度分多个平面，第0号平面采用4位十六进制，1号平面采用5位或6位，其他平面更长。第0号平面最重要，中文的范围为 `4E00-9FA5`。

> `\` 是为了转义，`一组`表示可能是多个十六进制
> javascript 允许直接采用unicode表示直接表示字符，这也就是为什么叫做 unicode 字符集

例子：`"\u0061" // "a"`

码点：unicode表示后面的十六进制就是码点，`0061`就是码点

### 2.3 utf-8编码

编码范围：所有语言，包括汉字、英文、韩文等

编码长度：可变长编码。常用 `英文字母 = 1Byte`，`汉字 = 3Byte`，生僻字符编码成4-6个字节

存储：跟unicode不同，采用可变长字节数存储，二进制

> 作为unicode的子集，2者存储方式不同

设计目的：扩展unicode，通过屏蔽位和移位操作实现快速读写，增加排序、查找等速度

> utf-8作为unicode的子编码

### 2.4 utf-16编码

编码范围：所有语言，包括汉字、英文、韩文等

编码长度：可变长编码，最少2byte

### 2.5 base64编码

base64 也是一种编码方式，并且广泛应用于 web 中。

1. 它是用 ASCII 码来表示二进制数据，也就是说 base64 的字符串只能是字母、数字、一些符号，不能包含中文
2. javascript 中，也可以说是浏览器实现了 `atob` 和 `btoa` 来解码和编码 base64 字符串
3. 采用 StringView 来实现普通字符串与 base64 字符串之间的转换，用于直接操作二进制数据

## 3 javascript 中的编码

官方：javascript 采用 unicode 字符集，utf-16 编码

问题：但是我们平时开发代码，编辑器右下又是 utf-8 编码，这个是怎么回事？

### 3.1 fromCharCode、fromCodePoint

`String.fromCharCode`：unicode字符(它的10进制表示法) -> 字符串

`String.fromCharCode(65,66,67) //A, B, C`

`String.fromCodePoint`: unicode字符(它的10进制表示法) -> 字符串 ，比 fromCharCode 更全面，es6新增的

### 3.2 charAt、charCodeAt、codePointAt

`String.prototype.charAt` : 返回字符串中索引字符 `"hello".charAt(0) //h`

`String.prototype.charCodeAt` ：返回字符串中索引字符对应的unicode字符 `"ABC".charCodeAt(0) //65`

`String.prototype.codePointAt` ：返回字符串中索引字符对应的unicode字符 `"ABC".codePointAt(0) //65`，比 charCodeAt 更全面，es6新增的，`'𠮷'.codePointAt(0).toString(16) //20bb7`

> 这里的65是十六进制的10进制表示法，转化成16进制应该是41，因为unicode采用的是16进制表示法。这一点在mdn上没有明确说出来，需要注意

```javascript
'A'.codePointAt(0) //65
'A'.codePointAt(0).toString(16) //41
'\u0041' // A
'\u{41}' // A
```

> codePointAt 与 fromCodePoint 互补

### 3.3 decodeURI、encodeURI、decodeURIComponent、encodeURIComponent

> javascript 中的转义字符，采用这2者进行转义或者反转义

`encodeURI` ：对URI整体编码，替换所有部分字符，不能被替换的保留字符 `& + = #` 等，因为它会认为这些特殊字符有特殊意义

`encodeURIComponent` ：对URI整体编码，用1到4个转义序列来表示这些需要编码的字符，但是它能替换保留字符 `& + = #` 等。它会将参数当做uri的尾部参数进行编码，因为它认为这些都只是普通的 text 字符，不具备特殊意义

2者相同点: 都是对URI编码

2者不同点

1. 编码范围不同(准确说对uri保留字符是否编码)：   
encodeURI 不会对 `; , / ? : @ & = + $ 字母 数字 - _ . ! ~ * ' ( ) #` 编码，会对 `中文` 进行编码  
encodeURIComponent 不会对 `字母、数字、(、)、.、!、~、*、'、-和_` 编码，会对 `中文 & + = #` 进行编码  
所以 encodeURIComponent 编码范围更广

2. 应用场景：如果需要在编码后，再使用这个url，那么使用 encodeURI ，如果需要对 uri 的参数进行编码，使用 encodeURIComponent

问：他们是将什么格式的编码转义成什么格式的编码？  
答：对一些特殊字符进行转义，也就是特殊字符才会从 utf-16 -> utf-8

问：我们用 encodeURIComponent 来干嘛？什么场景会用他  
答：encodeURIComponent 是用来将一些特殊字符转移成统一的 utf-8 编码，也就是将 utf-16 -> utf-8 编码，用一些转义字符替代utf-8不能识别的字符

### 3.4 Unicode 正规化

`normalize`

> normalize方法不能识别中文

`'\u01D1'.normalize() === '\u004F\u030C'.normalize()` // `Ǒ`

```javascript
// 标准等价合成
'\u004F\u030C'.normalize('NFC') // Ǒ
'\u004F\u030C'.normalize('NFC').length // 1
'\u004F\u030C'.normalize('NFC')[0] // Ǒ

// 标准等价分解
'\u01D1'.normalize('NFD') // Ǒ
'\u01D1'.normalize('NFD').length // 2
'\u01D1'.normalize('NFD')[0] // O
'\u01D1'.normalize('NFD')[1] // ˇ
```

### 3.5 escape、unescape

针对字符串，已经被废弃，使用 encodeURI、encodeURIComponent 替代

### 3.6 btoa、atob

> 主要用于base64操作，采用 StringView 替代

btoa： 将字符串(ASCII编码)转换成base64编码

atob： 将base64编码转换成字符串(ASCII编码)

window.btoa(string) 将 `string` 编码成 base64， 

1. base64 是用 ASCII 来表示
2. string 字符串，是 `二进制数据字符串`，每一个字符代表二进制数据的单个字节，及最多表示 2^8 = 256 个字符，所以一但字符不在区间 `0x00 - 0xFF` 内(unicode 16进制)，则会报错

## 4 中文编码

1. `GB2312` : ASCII + 简体中文，一个汉字占用2个字节，英文占1个字节 
2. `BIG5`： ASCII + 繁体中文，一个汉字占用2个字节，英文占1个字节
3. `GBK`：ASCII + 简体中文 + 繁体中文，一个汉字占用2个字节，英文占1个字节
4. `UTF-8`：万国码，占用字节可变长，一个汉字占3个字节
5. `GB18030`：ASCII + 简体中文 + 繁体中文 + 日文 + 朝鲜语等，占用字节可变长

问题：对一个url编码，如果只转义中文  
答：简单点的采用 `unescape(encodeURI(str))`；复杂一点，是将中文对应unicode码表区间做转义替换，也就是自己实现  
补充：encodeURI 能将中文、大括号、中括号进行转义，unescape 是恰好能将 encodeURI 转义的返回回去，只留下其他超出ASCII码的字符(包含中文)再次转义，mdn 上也没有写为什么可以这个样子

## 5 数组及字符串的 length

声明：array、string 的 `length` 属性，不代表我们常规理解的数组元素个数、字符串的元素个数

**array.length**: 表示数组中最大数字下标 + 1

```javascript
var arr = []
arr[1000] = 1
arr.length // 1001
```

解释：js中的数组就是对象，跟java或者其他类型语言不通，js没有数组这种数据结构。js中的数组就是对象，它的原型是 `Array.prototype`，`Array.prototype` 的原型就是 `Object.prototype`，即 `[].__proto__.__proto__ = Object.prototype`

> 所以在遍历数组的时候，最好使用 `for of` 或 `for in` 循环，因为它能正确识别元素个数

**string.length**: 表示字符串中的字符个数

> mdn: 该属性返回字符串中字符编码单元的数量

```javascript
'abc'.length // 3
'𠮷'.length // 2
```

解释：js 字符串的 length 属性的值如果不等于自身字符的个数，那么它必然存在Unicode码点大于0xFFFF的字符，因为 js 采用 utf-16编码规则，每个字符占2个字节，超出 0xFFFF 的字符，js 默认它占2个字符，也就是4个字节

> 所以在遍历字符串的时候，最好使用 `for of` 循环，因为它能正确识别占双字符的元素

## 参考文章

[百度百科 - 字符集](https://baike.baidu.com/item/%E5%AD%97%E7%AC%A6%E9%9B%86)  
[百度百科 - 字符编码](https://baike.baidu.com/item/%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81/8446880)
[mdn base64 的编码解码](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowBase64/Base64_encoding_and_decoding)
