# npm源码剖析系列-Schema校验神器之Yup


# npm源码剖析系列-Schema校验神器之Yup

## 一句话概括

yup是一个schema校验工具库。

前端用户表单的时候，我们经验需要在前后端校验用户的输入。一般而言，因为我们预先知道每个字段的含义，所以可以通过预先针对每一个字段执行条件语句判断是否合法。

1) 但是随着表单字段的增多，结构的复杂，校验规则就会变得复杂起来。如果遇到嵌套数组等情况，校验工作会变得更加复杂棘手。

2) 此外，如果我们的校验逻辑和业务逻辑耦合在一起，当校验规则需要调整的时候，需要进行大量的修改和回归操作。

因此，将校验工作抽离为单独的组件，并且支持灵活的配置规则，就变得特别重要。那yup的出现，就在于解决这些问题。

yup的使用非常简单，举个例子:

```jsx
let schema = yup.object().shape({
  name: yup.string().required(),
  age: yup.number().required().positive().integer(),
  email: yup.string().email(),
  website: yup.string().url(),
  createdOn: yup.date().default(function () {
    return new Date();
  }),
});

const validated = schema.validateSync({
  name: 'Hulk',
  age: 18,
});
```

上述schema就是一个对用户信息的校验逻辑，非常已于读取，可以很容易了解到各个字段的含义:

- name: 字符串，必须
- age: 数字、必须、整形、正数
- email: E-mail类型、可选
- website: Url类型、类型
- createdOn: 日期类型、可选、支持配置默认

当然实际上，yup支持的逻辑要复杂的多，具体在源码中详细分析。实际上还有一个非常常用好用的手段，可以基于yup，添加自己的校验逻辑，输出一个业务特色的字段校验库。

## 源码简析

相比之前的总结，Yup的源码变得复杂起来。从功能入手，Yup就是对用户输入进行Schema的校验。其核心代码也聚焦在这两个部分。

![](/images/202110/yup/yup_all.png)

两部分分别是: **构建/编译Schema** 和 **校验Schema**。

**构建** 是我们定义Schema格式的过程，参考我们文章开头的测试代码:

```jsx
let schema = yup.object().shape({
  name: yup.string().required(),
  age: yup.number().required().positive().integer(),
  email: yup.string().email(),
  website: yup.string().url(),
  createdOn: yup.date().default(function () {
    return new Date();
  }),
});
```

在上述代码中，我们定义一个用户信息的Schema校验规则。我们可以注意到这么几个细节:

1. yup的校验是支持对象，也支持简单的变量
2. yup的校验，支持可选和必选
3. yup内置了一些数字、字符串、日期等常见的校验格式，对于字符串等内置了常见的email、url等校验规则。
4. yup的schema构建，是支持 **链式调用** 的。

那么校验的过程呢，对比我们的测试代码:

```jsx
const validated = schema.validateSync({
  name: 'Hulk',
  age: 18,
});
```

因此对于校验的过程，validateSync是我们的核心切入点。在这里呢，我整理了一份整体的源码框架示意图，可以和大家一块，抓大放小的整理清楚。

### 整体流程
![](/images/202110/yup/yup_schema.png)

读懂了这张图，就能搞清楚Yup的核心思想。通过这个图，我们可以得出以下几个结论。

1. Yup的核心功能，均是通过Schema模块导出。而Schema也就是我们前面提到的构建Schema、校验Schema的基础，其物理含义则代表了一系列的规则。
2. Yup内置了一些常用的Schema，比如数字、字符串、对象、数组等等，足以覆盖我们常见的场景。
3. resolve和validate是其中比较重要的方法。

那么我们展开来看下。

### 构建Schema

**规则的存储**

Yup源码中，类Schema作为规则的核心承载，其源码在src/schema.ts。本小节我们主要看如何构建Schema。直接分析最复杂之一的 `object` 规则。

```jsx
// 创建Object规则
let schema = yup.object().shape({
  name: yup.string().required(),
  age: yup.number().required().positive().integer(),
});
```

**创建规则对象** 

Yup中内置了很对规则，通过 `object`、 `string` 、 `number` 等符号导出。导出的为一个构造方法，也就是当我们调用 `object` 的时候，本质上调用 `ObjectCrate`方法，并创建了一个对应的 `ObjectSchema`对象。相关代码如下

```jsx
// src/index.ts
export {
  mixedCreate as mixed,
  objectCreate as object, // 导出object方法
};

// src/object.ts
export function create<TShape extends ObjectShape>(spec?: TShape) {
  return new ObjectSchema<TShape>(spec) as OptionalObjectSchema<TShape>;
}
```

**校验validate** 

`validate` 是校验的主入口，这块代码有些复杂，我们先看父类 `BaseSchema` 的定义

```jsx
protected _validate(
    _value: any,
    options: InternalOptions<TContext> = {},
    cb: Callback,
  ): void {
    let value = _value;
    if (!strict) {
      // this._validating = true;
      value = this._cast(value, { assert: false, ...options });
      // this._validating = false;
    }
    let initialTests = [];

    if (this._typeError) initialTests.push(this._typeError);

    let finalTests: any[] = [];
    if (this._whitelistError) finalTests.push(this._whitelistError);
    if (this._blacklistError) finalTests.push(this._blacklistError);

    runTests(
      {
        args,
        value,
        path,
        sync,
        tests: initialTests,
        endEarly: abortEarly,
      },
      (err) => {
        if (err) return void cb(err, value);

        runTests(
          {
            tests: this.tests.concat(finalTests),
            args,
            path,
            sync,
            value,
            endEarly: abortEarly,
          },
          cb,
        );
      },
    );
  }
```

看着似懂非懂，其实整体上可以分为以下3步:

1. 整理数据，比如上述就整理了 `initialTests` 、 `finalTests`
2. 调用runTests，先验证 `initialTest` 规则是否满足；如果不满足，直接error callback.
3. 构建新的tests集合， `tests = this.tests.concat(finalTest)` ，然后重复步骤2

可以看出来，关键有两个元素，一个是测试集合 `tests`，另一个是 `runTest` 方法。 `runTest` 方法比较简单，不在赘述，重点看下 `tests` 测试集合如何生成。

通过 `BaseSchema` 代码可知， `test`为一个数组，当某个具体的Type在定义某些边界约束的时候，会操作 `test` 变量，具体 `boolean` 类型的例子

```jsx
isTrue(
    message = locale.isValue,
  ): BooleanSchema<TType | true, TContext, true | Optionals<TOut>> {
    return this.test({
      message,
      name: 'is-value',
      exclusive: true,
      params: { value: 'true' },
      test(value) {
        return isAbsent(value) || value === true;
      },
    }) as any;
  }
```

观察上述代码，我们可以通过 `yup.bool().isTrue()` 来引用。我们可以看到，其调用了 `BaseSchema` 的 `test` 方法，相关一些关键的实现如下

```jsx
test(...args: any[]) {

		if (typeof opts.test !== 'function')
      throw new TypeError('`test` is a required parameters');
    
    let next = this.clone();
    let validate = createValidation(opts);

    next.tests = next.tests.filter((fn) => {
      if (fn.OPTIONS.name === opts.name) {
        if (isExclusive) return false;
        if (fn.OPTIONS.test === validate.OPTIONS.test) return false;
      }
      return true;
    });

    next.tests.push(validate);

    return next;
  }
```

这下就清楚了，当子类调用 `this.test` 方法的时候，会重新构建整个test链：

1. clone一份当前的test集合
2. 遍历test数组，重新生成可以调用的test序列
3. 将入参新的test测试方法放入队尾
4. 返回整个test数组

**小结** 

至此，已经将关键的节点链路分析完毕，有些松散，也有些凌乱，我们重新梳理一遍。通过代码可以得知，yup在处理的时候，可以分为两类:

- 基本类型，比如 `boolean` `array` `number` `string` 等，这些校验比较简单，基本上是在类型判断的基础之上，添加一些边界的约束
- Object/Mix等复杂类型，基本上顺延基类的思路，但是关键的 `validate` 等方法都重新进行了设计。

## 如何使用

### 依赖其他字段不同

```jsx
const yup = require('yup');

let personSchema = yup.object({
  name: yup.string().min(4),
  gender: yup.string(),
  id: yup.string().when('gender', {
    is: 'F',
    then: s => s.matches(/F.*/g),
    otherwise: s => s.matches(/M.*/g)
  })
});

console.log(personSchema.isValidSync({
  name: 'Lucy',
  gender: 'F',
  id: 'F123',
}));
// true

console.log(personSchema.isValidSync({
  name: 'Lucy',
  gender: 'F',
  id: 'M123',
}));
// false

console.log(personSchema.isValidSync({
  name: 'John',
  gender: 'M',
  id: 'M123',
}));
// true
```

如上，我们定义了一个PersonSchema的校验规则，其中需要注意的是Id，通过when方法，根据gender性别的不同，定义了两种前缀。如果为女性F，那么要求id以F开头，如果为男性M，则要求id以M开头。

### 自定义String类型的判断规则

**方法1**: 通过正则匹配 

```jsx
let personSchema = yup.object({
  name: yup.string().min(4),
  gender: yup.string().matches(/F.*/g),
});

```

**方法2** 通过test方法匹配

```jsx
const yup = require('yup');

let personSchema = yup.object({
  name: yup.string().min(4),
  gender: yup.string(),
  id: yup.string().test('id', value => {
    return value.startsWith('F');
  })
});

console.log(personSchema.isValidSync({
  name: 'Lucy',
  gender: 'F',
  id: 'F123',
}));

console.log(personSchema.isValidSync({
  name: 'Lucy',
  gender: 'F',
  id: 'M123',
}));
```

### 异步判断

需要注意的，test方法传入的方法，支持异步的Async方法，在特殊情况下可以使用。比如以下代码

```jsx
const yup = require('yup');

let personSchema = yup.object({
  name: yup.string().min(4),
  gender: yup.string(),
  id: yup.string().test('id', async value => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (value.startsWith('F')) {
          resolve();
        } else {
          reject();
        }
      }, 1000)
    })
  })
});

personSchema.isValid({
  name: 'Lucy',
  gender: 'F',
  id: 'F123',
}).then(result => {
  console.log(result);
})

personSchema.isValid({
  name: 'Lucy',
  gender: 'F',
  id: 'M123',
}).then(result => {
  console.log(result);
})
```

### 判断其中子元素

```jsx
let schema = object({
  foo: array().of(
    object({
      loose: boolean(),
      bar: string().when('loose', {
        is: true,
        otherwise: (s) => s.strict(),
      }),
    }),
  ),
});

let rootValue = {
  foo: [{ bar: 1 }, { bar: 1, loose: true }],
};

await schema.validateAt('foo[0].bar', rootValue); // => ValidationError: must be a string

await schema.validateAt('foo[1].bar', rootValue); // => '1'
```

### 输出详细的错误信息

```jsx
import { setLocale } from 'yup';

setLocale({
  // use constant translation keys for messages without values
  mixed: {
    default: 'field_invalid',
  },
  // use functions to generate an error object that includes the value from the schema
  number: {
    min: ({ min }) => ({ key: 'field_too_short', values: { min } }),
    max: ({ max }) => ({ key: 'field_too_big', values: { max } }),
  },
});

// now use Yup schemas AFTER you defined your custom dictionary
let schema = yup.object().shape({
  name: yup.string(),
  age: yup.number().min(18),
});

schema.validate({ name: 'jimmy', age: 11 }).catch(function (err) {
  err.name; // => 'ValidationError'
  err.errors; // => [{ key: 'field_too_short', values: { min: 18 } }]
});
```

## What's more

Yup在设计的时候，还包括了很多增强的功能，最典型的比如cast数据转换等等。但是实际上在使用的时候，我们更聚焦在数据的校验，如果校验之后不合法，需要转化，需要处理，一般会有其他的业务逻辑负责。

把上面的原理大体读懂，常用的例子搞明白，就能掌握Yup。
