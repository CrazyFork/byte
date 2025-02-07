
#Notes

这个库的主要作用是用作协议在网络里边的编码. 比如 js - java 之间的网络交流通信.
其实如果简单来说一切都用json的str形式来就可以了. 但是就集团来说, 或者这个库的作者
来说, 是为了追求编码/解码/网络带宽的效率. 所以采用了了这种形式.

读这个库的起因还是由于公司用的node实现的hsf client底层用到了ta :( .

`test/main.java` 演示了java端的bytes的encoding

读这个库, 将收获到

* 如何在网络中编码/解码 int/long/string 类型
* 如何用CESU-8编码, 将js中的UTF-16字符编码转换成utf8兼容的编码格式的一个转换流程
* 操作bytes和buffer

## tips

->  在理解下面这种转换, 用python脚本
`this._bytes[index++] = (0xc0 + ((ch >> 6) & 0x1f)) >>> 32;`

```python
"{0:b}".format(0x3f)

# output: 
# '111111'
```

-> utf8 从bytes中读取

```
  return this._bytes.toString('utf8', index, index + length);
```


-> `_putString` 和 `putRawString` 的区别是啥呀?

感觉从作用上来说都是往buffer中写入utf8 compatible 的 encoding. 为啥要写俩方法呀

* `_putString` 
  * node native 支持的, `Buffer.write(str, )` 就支持往 buffer 中 write utf8 encoded string. `this._bytes.write(value, valueOffset);`, 
  * `_putString` 会在前面写个4 bytes 的int, __然后才是utf8真正的string? 为啥要这样呢?__

  ```js
  // :write a 4 bytes of int of index ?
  this.putInt(index, length);
  const valueOffset = index + 4;

  if (isBuffer) {
    value.copy(this._bytes, valueOffset);
  } else {
    this._bytes.write(value, valueOffset);
  }

  if (format === 'c') {
    this.put(valueOffset + length, 0);
  }
  ```


  -> egg-bin 看来是默认支持了 mocha 的 unit-test, 项目中啥配置也没有`"test": "npm run lint && egg-bin test",`



# 其他

```javasscript

// long 的编码被拆解成了 2 个 4bytes 的数字的组合, 塞入到buffer里边的
  if (this._order === BIG_ENDIAN) {
    this._bytes.writeInt32BE(value.high, highOffset);
    this._bytes.writeInt32BE(value.low, lowOffset);
  } else {
    this._bytes.writeInt32LE(value.high, highOffset);
    this._bytes.writeInt32LE(value.low, lowOffset);
  }



// 从代码上来看是将 UTF-16 编码转换成了 UTF-8 相兼容的 CESU-8 的方式
  // CESU-8 Bit Distribution
  // @see http://www.unicode.org/reports/tr26/
  //
  // UTF-16 Code Unit                   | 1st Byte               | 2nd Byte               | 3rd Byte
  // 000000000xxxxxxx (0x0000 ~ 0x007f) | 0xxxxxxx (0x00 ~ 0x7f) |                        |
  // 00000yyyyyxxxxxx (0x0080 ~ 0x07ff) | 110yyyyy (0xc0 ~ 0xdf) | 10xxxxxx (0x80 ~ 0xbf) |
  // zzzzyyyyyyxxxxxx (0x0800 ~ 0xffff) | 1110zzzz (0xe0 ~ 0xef) | 10yyyyyy (0x80 ~ 0xbf) | 10xxxxxx (0x80 ~ 0xbf)

```








byte
=======

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Gittip][gittip-image]][gittip-url]
[![David deps][david-image]][david-url]
[![node version][node-image]][node-url]
[![npm download][download-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/byte.svg?style=flat-square
[npm-url]: https://npmjs.org/package/byte
[travis-image]: https://img.shields.io/travis/node-modules/byte.svg?style=flat-square
[travis-url]: https://travis-ci.org/node-modules/byte
[coveralls-image]: https://img.shields.io/coveralls/node-modules/byte.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/node-modules/byte?branch=master
[gittip-image]: https://img.shields.io/gittip/fengmk2.svg?style=flat-square
[gittip-url]: https://www.gittip.com/fengmk2/
[david-image]: https://img.shields.io/david/node-modules/byte.svg?style=flat-square
[david-url]: https://david-dm.org/node-modules/byte
[node-image]: https://img.shields.io/badge/node.js-%3E=_0.10-green.svg?style=flat-square
[node-url]: http://nodejs.org/download/
[download-image]: https://img.shields.io/npm/dm/byte.svg?style=flat-square
[download-url]: https://npmjs.org/package/byte

Input Buffer and Output Buffer,
just like Java [`ByteBuffer`](http://docs.oracle.com/javase/6/docs/api/java/nio/ByteBuffer.html).

## Install

```bash
$ npm install byte --save
```

## Usage

All methods just like Java ByteBuffer,
you find them out [here](http://docs.oracle.com/javase/6/docs/api/java/nio/ByteBuffer.html#method_summary).

```js
var ByteBuffer = require('byte');

var bb = ByteBuffer.allocate(1024);
bb.order(ByteBuffer.BIG_ENDIAN); // default is BIG_ENDIAN, you can change it to LITTLE_ENDIAN.
bb.put(0);
bb.put(new Buffer([0, 1, 2]));
bb.put(new Buffer([255, 255, 255, 255]), 10, 3);
bb.put(21, 100);

bb.putChar('a');
bb.putChar(10, 'b');

bb.putInt(1024);
bb.putInt(-100);

bb.putFloat(100.9);

bb.putLong(10000100009099);
bb.putLong('1152921504606847000');

bb.putShort(65535);
bb.putShort(-50000);

bb.putDouble(99.99999);

// wrap for read
var rb = ByteBuffer.wrap(new Buffer(100));
rb.getInt();
rb.getLong();
rb.getChar();
rb.get();
rb.getDouble();
rb.getFloat();
```

## Benchmark

```bash
$ node benchmark/put.js

node version: v0.11.12, date: Mon May 12 2014 18:25:35 GMT+0800 (CST)
Starting...
20 tests completed.

put()                                      x 29,971,599 ops/sec ±4.10% (96 runs sampled)
putChar("a")                               x 27,950,189 ops/sec ±6.22% (80 runs sampled)
putChar(61)                                x 34,798,492 ops/sec ±5.08% (81 runs sampled)
putShort()                                 x 25,264,781 ops/sec ±2.90% (88 runs sampled)
putInt()                                   x 21,368,588 ops/sec ±6.07% (85 runs sampled)
putFloat()                                 x 12,324,148 ops/sec ±2.04% (93 runs sampled)
putDouble()                                x 13,374,686 ops/sec ±1.41% (92 runs sampled)
putLong(100000)                            x 17,754,878 ops/sec ±5.16% (86 runs sampled)
putSmallSLong("10000")                     x  7,732,989 ops/sec ±2.07% (92 runs sampled)
putBigNumLong(34359738368)                 x  3,580,231 ops/sec ±2.58% (93 runs sampled)
putSafeStrLong("34359738368")              x  2,443,560 ops/sec ±2.04% (97 runs sampled)
putStrLong("9223372036854775808")          x    760,908 ops/sec ±2.42% (92 runs sampled)
ByteBuffer.allocate(100).putString(0, str) x    608,403 ops/sec ±11.46% (70 runs sampled)
putString(0, str)                          x  1,362,412 ops/sec ±8.55% (85 runs sampled)
bytes.putString(str)                       x  1,506,610 ops/sec ±2.31% (94 runs sampled)
putString(0, buf)                          x  5,947,594 ops/sec ±4.16% (90 runs sampled)
bytes.putString(buf)                       x  5,741,251 ops/sec ±1.69% (95 runs sampled)
putRawString(0, str)                       x  2,908,161 ops/sec ±1.81% (95 runs sampled)
bytes.putRawString(str)                    x  1,527,089 ops/sec ±4.98% (86 runs sampled)
bytes.putRawString(str).array()            x  1,009,026 ops/sec ±2.38% (91 runs sampled)

$node benchmark/get.js

node version: v0.11.12, date: Mon May 12 2014 19:14:26 GMT+0800 (CST)
Starting...
15 tests completed.

get(0, 1) => copy Buffer    x  2,059,464 ops/sec ±9.18% (69 runs sampled)
get(0, 100) => copy Buffer  x  2,124,455 ops/sec ±4.98% (75 runs sampled)
get(0, 4096) => copy Buffer x    356,927 ops/sec ±9.43% (56 runs sampled)
get() => byte               x 15,477,897 ops/sec ±3.05% (89 runs sampled)
getChar(0)                  x 52,541,591 ops/sec ±1.04% (95 runs sampled)
getShort(0)                 x 26,297,086 ops/sec ±2.46% (89 runs sampled)
getInt(0)                   x 18,772,003 ops/sec ±6.27% (71 runs sampled)
getFloat(0)                 x 13,132,298 ops/sec ±1.68% (97 runs sampled)
getDouble(0)                x 10,968,594 ops/sec ±1.27% (94 runs sampled)
getLong(0)                  x 11,849,374 ops/sec ±2.63% (96 runs sampled)
getString(0)                x  2,358,382 ops/sec ±5.78% (76 runs sampled)
getCString(0)               x  1,618,356 ops/sec ±8.41% (72 runs sampled)
readRawString(4, 100)       x  4,790,991 ops/sec ±9.25% (79 runs sampled)
readRawString(100)          x  5,434,663 ops/sec ±1.32% (95 runs sampled)
getRawString(0, 100)        x  5,497,325 ops/sec ±1.02% (98 runs sampled)
```

## `Number` methods

```
putShort / putInt16
putUInt16
putInt / putInt32
putUInt / putUInt32
putInt64
putFloat
putDouble


getShort / getInt16
getUInt16
getInt / getInt32
getUInt / getUInt32
getInt64
getFloat
getDouble
```

## `String` methods

Java String format: `| length (4 bytes int) | string bytes |`

C String format: `| length + 1 (4 bytes int) | string bytes | \0 |`

Row String format: `string bytes`

### `putString()` and `putCString()` and `putRawString()`

```js
bb.putString('foo');
bb.putString(new Buffer('foo'));

bb.putCString('foo');
bb.putCString(new Buffer('foo'));

bb.putRawString('foo');
```

### `getString()` and `getCString()` and `getRawString(), readRawString()`

```js
bb.getString();
bb.getString(10);

bb.getCString();
bb.getCString(10);

bb.getRawString(0, 10);
bb.readRawString(10);
```

## License

(The MIT License)

Copyright (c) 2013 - 2014 fengmk2 &lt;fengmk2@gmail.com&gt; and other contributors

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
