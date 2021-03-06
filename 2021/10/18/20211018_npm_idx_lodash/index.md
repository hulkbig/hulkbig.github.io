# npm源码剖析系列-防御式编程利器: lodash.get 和 idx


![Banner](/images/202110/npm_idx_lodash/banner.png)

今天和大家聊聊两个常用的库
* lodash.get https://www.npmjs.com/package/lodash.get
* Idx https://www.npmjs.com/package/idx

## Can not read property of undefined
取不到应该有的值，是开发中常见的情况，大部分语言都有这个问题，C语言会变成野指针；Java会有NullPointerException；同样的在JavaScript中就是undefined。在和其他三方库或者服务端交互的时候，这种情况经常出现。一旦出现，就很糟心，会阻塞代码执行；会抛出异常；甚至会引起白屏等渲染异常。

图片

为了避免我们的代码挂掉，我们常常会采用防御式编程。当然这是文绉绉的说法，简单来说，就是一路判断下去，比如:

```javascript
variables && variable.propertyA && variable.propertyA.propertyB.
```

数据结构简单的情况下，这样写挺好；可是如果层数特别多，又或是有数组，情况就变得难受起来，举个例子:

```javascript
variables && variable.propertyA && variable.propertyA.propertyB && variable.propertyA.propertyB[0] && variable.propertyA.propertyB[0].propertyC
```

## 怎么省事？
lodash.get和idx这两个库都可以帮我们面对这种情况，先看lodash.get.

lodash.get实际上是lodash工具库的一个子集。这是一些官方的例子。

```javascript
var object = { 'a': [{ 'b': { 'c': 3 } }] };
_.get(object, 'a[0].b.c');
// => 3
_.get(object, ['a', '0', 'b', 'c']);
// => 3
_.get(object, 'a.b.c', 'default');
// => 'default'
```
`_.get(object, path, [defaultValue])` 有3个变量，
1. 为被索引的对象object
2. 为访问属性的路径path
3. 为默认数值defaultValue。


其中关键的参数为path。举个例子，如果path为`a[0].b.c`，则表示a为数组，a的第一个元素，并进一步获取该其下的属性b，属性b下的属性c。那么我们把前面的例子用lodash.get重新写一遍，怎么写呢？

```javascript
// pure javascript
variables && variable.propertyA && variable.propertyA.propertyB && variable.propertyA.propertyB[0] && variable.propertyA.propertyB[0].propertyC
```
```javascript
// lodash.get
_.get(variables, ‘propertyA.propertyB[0].propertyC’)
```
简单明了。同样的，idx的作用是一样。

```javascript
// idx
idx(variables, _ => _.propertyA.propertyB[0].propertyC)
```


哪个更好呢？从下载量上看，lodash.get的使用人数会更多，但是两者功能是同样的，性能也没大差别，看个人的喜好即可。

![Banner](/images/wechat.png)

