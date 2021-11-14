# validator 字符串判断神器

# validator 字符串判断神器

## 一句话概括

我们经常需要判断一个字符串，是不是符合某种业务规则。比如是否是E-mail? 电话？信用卡？等等。很多情况下，我们会手写正则去判断，但是实际上经常会遗漏各种边界条件。

validator就可以帮我们解决这种问题。内置了几十种的判断方法，简单列几个，可以按需索取。

- isAfter
- isAlpha
- isAscii
- isBase64
- isDate
- ...

## 安装和使用

安装

```javascript
npm i validator
```

使用

```javascript
var validator = require('validator');

validator.isEmail('foo@bar.com'); //=> true
```

## 源码简析

validator，就是对字符串进行一系列的规则校验。根据每个业务场景的不一样，所以每个规则的校验方法也不同。其中呢，大部分是通过正则表达式的校验，因此在其中有些判断逻辑，甚至是正则表达式我们可以好好收藏一下。

### 判断是字符串

一般情况下，判断字符串只要看看是不是string就可以了，如果要做的完美一些了，还要看看是不是String的对象。 `typeof new String() === 'object'`

```jsx
export default function assertString(input) {
  const isString = typeof input === 'string' || input instanceof String;

  if (!isString) {
    let invalidType = typeof input;
    if (input === null) invalidType = 'null';
    else if (invalidType === 'object') invalidType = input.constructor.name;

    throw new TypeError(`Expected a string but received a ${invalidType}`);
  }
}
```

### 判断是否为Boolean

看到这个问题，你可能和我一样吃惊，但是当你真的遇到各种情况下boolean的实际值，更会惊吓下巴。

```jsx
const defaultOptions = { loose: false };
const strictBooleans = ['true', 'false', '1', '0'];
const looseBooleans = [...strictBooleans, 'yes', 'no'];

export default function isBoolean(str, options = defaultOptions) {
  assertString(str);

  if (options.loose) {
    return looseBooleans.includes(str.toLowerCase());
  }

  return strictBooleans.includes(str);
}
```

可以注意的是，很多时候 `1`和 `true`都代表了布尔值 `true` ，同样的 `0`和 `false`都代表了布尔值 `false` ，尤其在序列化反序列化的场景下，经常会遇到。

## 常用的API

[https://github.com/validatorjs/validator.js](https://github.com/validatorjs/validator.js)

![Banner](/images/wechat.png)

