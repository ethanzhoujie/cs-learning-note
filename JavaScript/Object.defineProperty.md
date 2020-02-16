# Object.defineProperty的用法详解

该方法是es5的方法（千万不要以为是es6的哦），作用是直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。（**切记只能用在对象身上不能用在数组身上**）

### 1、语法

```
Object.defineProperty(obj, prop, descriptor)复制代码
```

### 2、参数说明：

- obj：必需。目标对象 
- prop：必需。需定义或修改的属性的名字
- descriptor：必需。目标属性所拥有的特性

### 3、属性使用案例

修改某个属性的值时，给这个属性添加一些特性。

```
let person = {}; 
Object.defineProperty(person, 'name', {   
    writable: true || false,   
    configurable: true || false,   
    enumerable: true || false,  
    value:'gjf' 
});
复制代码
```

属性详解：

- writable：是否可以被重写，true可以重写，false不能重写，默认为false。
- enumerable：是否可以被枚举（使用for...in或Object.keys()）。设置为true可以被枚举；设置为false，不能被枚举。默认为false。
- value：值可以使任意类型的值，默认为undefined
- configurable：是否可以删除目标属性或是否可以再次修改属性的特性（writable, configurable, enumerable）。设置为true可以被删除或可以重新设置特性；设置为false，不能被可以被删除或不可以重新设置特性。默认为false。

### 4、存取器描述（get和set）

- **注意：当使用了getter或setter方法，不允许使用writable和value这两个属性**

```
let person = {};
let n = 'gjf';
Object.defineProperty(person, 'name', { 
    configurable: true,  
    enumerable: true, 
    get() {    
        //当获取值的时候触发的函数    
        return n  
    },  
    set(val) {    
        //当设置值的时候触发的函数,设置的新值通过参数val拿到    
        n = val;  
    }
});
console.log(person.name) //gjf
person.name = 'newGjf'
console.log(person.name) //newGif
```







# 深入探究Object.defineProperty

一般情况下，我们给对象添加属性值都是这样子的：

```
let obj = {};
obj.key = 123;
复制代码
```

其实，还有一种添加属性的方式是：Object.defineProperty(obj, prop, descriptor)，这个API非常重要，它是VUE等框架实现数据响应式的基石。利用它设置的访问器属性可以做到数据劫持，然后再通过订阅/发布模式将数据的变化反应到视图上。

不扯远了，本文先来单纯地探究一下 Object.defineProperty

## 定义属性

```
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: false,
  enumerable: false,
  writable: false,
  value: 123
});
复制代码
```

我们称 configurable、enumerable、writable、value 为属性的四个描述符（即descriptor），也可以称它们为属性的四个特性：

### configurable

当且仅当该属性的 configurable 为 true 时，该属性的描述符才能够被改变，同时该属性也能从对应的对象上被删除。例子：

```
// 为true时可以改变其他描述符，也可以删除这个属性
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: true,
  enumerable: false,
  writable: false,
  value: 123
});
Object.defineProperty(obj, "key", {
  enumerable: true
});

// 为false时不可以改变其他描述符，也不能删除这个属性
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: false,
  enumerable: false,
  writable: false,
  value: 123
});
Object.defineProperty(obj, "key", {enumerable: true});      //报错
delete obj.key;     // false
复制代码
```

### enumerable

当且仅当该属性的enumerable为true时，该属性才能被枚举。

```
// 为true时属性可以被枚举
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: true,
  enumerable: true,
  writable: false,
  value: 123
});
for (var i in obj) {    
  console.log(i);       // key
}

// 为false时属性可以不被枚举
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: true,
  enumerable: false,
  writable: false,
  value: 123
});
for (var i in obj) {    
  console.log(i);      // undefined 
}
复制代码
```

### writable

当且仅当该属性的writable为true时，value才能被赋值运算符改变。

```
// 为true时属性可以被改变
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: true,
  enumerable: true,
  writable: true,
  value: 123
});
obj.key = 456;
console.log(obj.key);       // 456

// 为false时属性不可以被改变
let obj = {};
Object.defineProperty(obj, "key", {
  configurable: true,
  enumerable: true,
  writable: false,
  value: 123
});
obj.key = 456;
console.log(obj.key);       // 123
复制代码
```

### value

该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）

注意，如果用 Object.defineProperty 定义属性时没有主动设置描述符，那么所有的描述符的值为false。

```
我们用 Object.getOwnPropertyDescriptor(obj, "key") 来获取属性的描述符信息

// 例1
let obj = {};
Object.defineProperty(obj, "key", {}); 
Object.getOwnPropertyDescriptor(obj, "key");
<!--
    {
    configurable: false,
    enumerable: false,
    writable: false,
    value: undefined
}
-->

// 例2
let obj = {};
Object.defineProperty(obj, "key", {value: 123}); 
Object.getOwnPropertyDescriptor(obj, "key");
<!--
    {
    configurable: false,
    enumerable: false,
    writable: false,
    value: 123
}
-->

// 例3
let obj = {};
Object.defineProperty(obj, "key", {writable: true}); 
Object.getOwnPropertyDescriptor(obj, "key");
<!--
    {
    configurable: false,
    enumerable: false,
    writable: true,
    value: undefined
}
-->
复制代码
```

## 修改属性

对于一个已经存在的属性，它的默认描述符都为true。 因为与上文中的注意相悖，所以此处极容易混淆

```
let obj = {key: 123};
Object.getOwnPropertyDescriptor(obj, "key");
<!--
    {
    configurable: true,
    enumerable: true,
    writable: true,
    value: 123
}
-->
// 可以修改
Object.defineProperty(obj, "key", {writable: false});
Object.getOwnPropertyDescriptor(obj, "key");
<!--
    {
    configurable: true,
    enumerable: true,
    writable: false,
    value: 123
}
-->
```