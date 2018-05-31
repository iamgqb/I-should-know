

### typeof

用来返回表达式的类型

```javascript
typeof (unary expression)
```

可返回类型见规范: https://www.ecma-international.org/ecma-262/6.0/#sec-typeof-operator

其中需注意的是 `typeof null === 'object'`;

实现了 [[call]] 的 object 返回 function


#### js 基础数据类型
- Null
- Undefined
- Number
- String
- Boolean
- Symbol
- Object


### instanceof

用来检查 *A 的原型链* 中是否存在 *B.prototype*;

A instanceof B

其本质是循环获取 A.prototype 来与 B.prototype 做比较

参看规范: https://www.ecma-international.org/ecma-262/6.0/#sec-instanceofoperator

step 6 即循环获取步骤, 直到到达 最原始的Object

```javascript
Object.getPrototypeOf(A) === B.prototype ?

Object.getPrototypeOf(Object.getPrototypeOf(A)) === B.prototype ?

...

```

`Object.getPrototypeOf` 方法用来获取指定对象的 prototype


#### 特别的

`A.isPrototypeOf(B)` 检查 *A* 是否在 *B 的原型链*上, 

因此

`A.prototype.isPrototypeOf(B)` 可以检查 A.prototype 是否存在 B 的原型链上，与 instanceof 正好相反


#### example
```javascript

class A {}
class AA extends A {}

const aa = new AA();

aa instanceof AA === true;
aa instanceof A === true;

Object.getPrototypeOf(aa) === AA.prototype;
Object.getPrototypeOf(aa) !== A.prototype;
Object.getPrototypeOf(Object.getPrototypeOf(aa)) === A.prototype; // 感受一下

AA.prototype.isPrototypeOf(aa) === true; // 与 aa instanceof AA 相反
A.prototype.isPrototypeOf(aa) === true; // 与 aa instanceof A
A.isPrototypeOf(AA) === true;

```




