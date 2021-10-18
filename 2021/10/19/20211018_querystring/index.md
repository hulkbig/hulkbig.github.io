# URL的参数你真的懂吗？-querystring


# URL的参数你真的懂吗？-querystring

```html
https://example.com/path/to/page?name=ferret&color=purple
```

大家都熟悉，querystring是URL的参数传递方式。上面就是他的数据格式。

但是如果要问问你querystring到底来自哪个标准？具体的处理方式你真的懂吗？可能大部分人都要摇头。这篇文章就能帮到你。

## 背景

querystring主要来自URL标准，URL全称为Uniform Resource Locators，统一资源定位符。Web是其最常见的使用场景，除此之外，ftp、ssh等协议也是基于这些标准来使用的，举个例子。

```html
# Web URL
https://example.com/path/to/page?name=ferret&color=purple

# Ftp URL
ftp://test:test@192.168.0.1:21/profile
```

URL是一个标准，主要有两个版本:

rfc1738: [https://datatracker.ietf.org/doc/html/rfc1738](https://datatracker.ietf.org/doc/html/rfc1738)

rfc3986: [https://datatracker.ietf.org/doc/html/rfc3986](https://datatracker.ietf.org/doc/html/rfc3986)

querystring就来自这个URL标准，querystring主要处理两部分:

1. 数据传输格式:  `paramA=valueA&paramB=valueB`
2. 数据的编码解码: 1)预留字符 # :  2) 特殊字符转码 `SPACE -> 0x20` 

qs库呢，就是这两部分逻辑的具体实现。那么刚刚提到了rfc1738和rfc3986，在querystring方便主要是对 `SPACE %20`  的处理方式不同，rfc1738 `%20-> +`, rfc3986保留 `%20`

**数据传输格式**

```html
field1=value1&field2=value2&field3=value3...
```

当然可能有的场景也会传输一些特殊的字符，比如JSON、数组等等，但是一般大家在使用过程中，都会先encode，变成普通的字符串；解码的时候再进行反向操作。

**数据的编码解码**

我们不去读常常的RFC标准了，在Wiki上可以找到如下的编解码定义:

1. Characters that cannot be converted to the correct charset are replaced with HTML numeric character references[11]
2. SPACE is encoded as '+' or '%20'
3. Letters (A–Z and a–z), numbers (0–9) and the characters '~','-','.' and '_' are left as-is
4. + is encoded by %2B
5. All other characters are encoded as a %HH hexadecimal representation with any non-ASCII characters first encoded as UTF-8 (or other specified encoding)


用大白话呢就是下面这个，注意步骤2中关于SPACE空格的处理，就是两个不同协议的区别。

1. 如果某个字符编码不能转成对应的字符，那么就使用数字的替代表示(数学字符等)
2. 空白SPACE，转成 + 或者 %20 ( + 的ASCII编码为 0x20)
3. A-z a-z 0-9 ~ - . _ 这些字符保留
4. + 转为 %2B
5. 其他剩余字符，转为%HH的十六进制表示

### qs

回到qs这个依赖库，就是对上述操作进行了封装。当然qs本身在对象数据的传输、解析(深度控制等)、数组等做了很多工作，但是实际上用的都比较少。

**使用** 

举个官方的例子

```jsx
var qs = require('qs');
var assert = require('assert');

var obj = qs.parse('a=c');
assert.deepEqual(obj, { a: 'c' });

var str = qs.stringify(obj);
assert.equal(str, 'a=c');
```

**What's More**

在面向IE等老破浏览器编程的时候，其实这种库是特别有价值的，但是现代浏览器和Node都提供了相似的内置API，可以直接使用

Web:  [https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams)

Node: [https://nodejs.org/api/url.html#url_class_urlsearchparams](https://nodejs.org/api/url.html#url_class_urlsearchparams)

![Banner](/images/wechat.png)

