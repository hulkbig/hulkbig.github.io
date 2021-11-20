# 你一定要知道的glob


## 是什么？什么时候用？

glob是npm环境下的一个glob实现，glob是通过通配符来匹配识别一系列文件的工具。比如想匹配当前目录下的所有js文件，可以非常简单的通过以下方法获取:

```jsx
glob("**/*.js", (err, files) => {
  console.log(JSON.stringify(files));
})
```

如果不适用glob，那就需要写特别复杂的正则工具，费事费力，还容易出现问题。所以glob是Node下一个非常重要的工具，我们都应该掌握。

## 安装和使用

### 安装和使用

**安装** 

```bash
npm i glob
```

**使用**

```jsx
var glob = require("glob")

// options is optional
glob("**/*.js", options, function (er, files) {
})
```

glob工具本身的用法非常简洁，其实掌握这个工具的重点在于学会glob本身的规则，那接下来就一起继续。

## glob

前面提到，glob起源于Unix，通过通配符的方式去匹配文件，通配符的掌握也是理解glob的重点。一般而言，主要需要掌握以下的语法：

| Pattern | 含义 |
| :-----| :-----|
| * | 匹配任何字符(包括空白) |
| ？ | 匹配一个字符 |
| [...] | 类似正则表达式，匹配一系列字符; 如果第一个字符为!/^则表示匹配取反。 |
| !(pattern\|pattern\|pattern) | 任何不命中patterns |
| ?(pattern\|pattern\|pattern) | 任何只命中1个patterns |
| +(pattern\|pattern\|pattern) | 命中1个或者多个pattern |
| *(a\|b\|c) | 匹配0个或者多个pattern，没有什么实际意义 |
| @(pattern\|pat*\|patterN) | 匹配其中任何一个pattern |
| ** | 匹配中间任意子目录 |
| dot(.) | 如果pattern中有.作为起始符，那么默认情况下，glob的表达式不会生效，除非在表达式中也有点号.作为其开始。 |

### 如何方便的测试glob

就像开发正则一样，我们需要对设计的glob的表达式进行测试，哪种方法最为常见呢？

```bash
echo *.js
```

没错，只要打开Terminal，输入 `echo *.js` 就可以显示当前目录下的所有js文件。无需额外以来，也不需要额外的编码。 

**几个glob的例子**

1. 匹配当前目录下的js文件 `*.js`
2. 匹配当前目录及其子目录下所有的js文件 `**/*.js`
3. 匹配子目录下所有js文件 `./*/**/*.js`

![Banner](/images/wechat.png)

