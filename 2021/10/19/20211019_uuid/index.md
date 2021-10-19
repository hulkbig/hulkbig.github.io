# UUID到底是什么？npm分享之uuid


## npm仓库信息

npm: [https://www.npmjs.com/package/uuid](https://www.npmjs.com/package/uuid)
downloads:  460w
git: [github.com/uuidjs/uuid#readme](http://github.com/uuidjs/uuid#readme)
repo: 12k

## UUID分析
UUID(Universally unique identifier)，统一资源定位符，可以用来标识唯一的资源。典型的UUID长这个样子。
```jsx
123e4567-e89b-12d3-a456-426614174000
xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx
```

你可能经常使用，但是大概率很难讲个所以然。实际上，目前我们使用的UUID，主要遵循RFC4122版本，协议的链接在 [https://www.ietf.org/rfc/rfc4122.txt](https://www.ietf.org/rfc/rfc4122.txt) 。协议的本身有很多个版本，比如v1，v2，v3，v4，v5。版本不同是因为其使用的工况可能不同，官方的定义如下:

```
   Msb0  Msb1  Msb2  Msb3   Version  Description

    0     0     0     1        1     The time-based version
                                     specified in this document.

    0     0     1     0        2     DCE Security version, with
                                     embedded POSIX UIDs.

    0     0     1     1        3     The name-based version
                                     specified in this document
                                     that uses MD5 hashing.

    0     1     0     0        4     The randomly or pseudo-
                                     randomly generated version
                                     specified in this document.

    0     1     0     1        5     The name-based version
                                     specified in this document
                                     that uses SHA-1 hashing.
```

版本多的让人眼花缭乱，那么实际上在使用的时候改用哪种呢？参考StackOverflow的高票答案，可以按照这个方法选择

- 仅需要一个UUID: 可以使用version1 和 version4
    - version1: 基于网卡mac地址+时间戳构建。
    - version4: 完全基于随机数/伪随机数构建。
- UUID需要基于一定的参数可重建(reproducible)，需要version3或者version5
    - version3: 基于namespace+name → MD5
    - version5: 基于namespace+name → SHA-1

基于以上的原则，就可以容易的选择版本。其中有个细节可以注意的是，由于v4版本完全基于时间戳构建，所以理论上可能存在UUID碰撞的可能。但是实际在使用中不用太在意，有两个原因: 1. 碰撞的几率天生就很小; 2. 碰撞几率计算也是根据所有场景放在一起，但是每个应用都有自己的场景，uuid也不会通用，因此实际上不用担心。

当然，如果只是使用uuidv4的话，可以使用这个代码片段 ([https://stackoverflow.com/questions/105034/how-to-create-a-guid-uuid](https://stackoverflow.com/questions/105034/how-to-create-a-guid-uuid))

```jsx
function uuidv4() {
  return ([1e7]+-1e3+-4e3+-8e3+-1e11).replace(/[018]/g, c =>
    (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
  );
}

console.log(uuidv4());
```

## 使用

```jsx
import { v1 as uuidv1 } from 'uuid';

uuidv1(); // ⇨ '2c5ea4c0-4067-11e9-8bad-9b1deb4d3b7d'
```

使用起来比较简单，不在赘述。

## What'more

无三方库。

![Banner](/images/wechat.png)

