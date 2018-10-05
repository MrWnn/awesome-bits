# awesome-bits [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/sindresorhus/awesome)

> 🐂🍺位运算技巧一览表
>
> 维护者 - [Keon Kim](https://github.com/keonkim)
> 欢迎[pull requests](https://github.com/keonkim/awesome-bits/pulls)




## 整数
**设置第n位（置为1）**
```
x | (1<<n)
```
**取消第n位（置为0）**
 ```
 x & ~(1<<n)
 ```
**切换第n位**
```
x ^ (1<<n)
```
**向上取整至2的幂**
```
unsigned int v; //only works if v is 32 bit
v--;
v |= v >> 1;
v |= v >> 2;
v |= v >> 4;
v |= v >> 8;
v |= v >> 16;
v++;
```
**向下取整**
```
n >> 0

5.7812 >> 0 // 5

```
**取最大int值**
```
int maxInt = ~(1 << 31);
int maxInt = (1 << 31) - 1;
int maxInt = (1 << -1) - 1;
int maxInt = -1u >> 1;
```
**取最小int值**
```
int minInt = 1 << 31;
int minInt = 1 << -1;
```
**取最大long值**
```
long maxLong = ((long)1 << 127) - 1;
```
**乘以2**
```
n << 1; // n*2
```
**除以2**
```
n >> 1; // n/2
```
**乘以2的m次幂**
```
n << m;
```
**除以2的m次幂**
```
n >> m;
```
**检查相等**

<sub>*在Javascript中快35%*</sub>
```
(a^b) == 0; // a == b
!(a^b) // use in an if
```
**检查奇数**
```
(n & 1) == 1;
```
**交换两个值**
```
//version 1
a ^= b;
b ^= a;
a ^= b;

//version 2
a = a ^ b ^ (b = a)
```
**取绝对值**
```
//version 1
x < 0 ? -x : x;

//version 2
(x ^ (x >> 31)) - (x >> 31);
```
**取二者最大值**
```
b & ((a-b) >> 31) | a & (~(a-b) >> 31);
```
**取二者最小值**
```
a & ((a-b) >> 31) | b & (~(a-b) >> 31);
```
**检查符号一致性**
```
(x ^ y) >= 0;
```
**符号取反**
```
i = ~i + 1; // or
i = (i ^ -1) + 1; // i = -i
```
**计算2<sup>n</sup>**
```
1 << n;
```
**是否为2的幂**
```
n > 0 && (n & (n - 1)) == 0;
```
**m对2<sup>n</sup>取模**
```
m & ((1 << n) - 1);
```
**取平均值**
```
(x + y) >> 1;
((x ^ y) >> 1) + (x & y);
```
**取n的第m位（由低到高）**
```
(n >> (m-1)) & 1;
```
**置n的第m位为0（由低到高）**
```
n & ~(1 << (m-1));
```
**检查第n位是否为1**
```
if (x & (1<<n)) {
  n-th bit is set
} else {
  n-th bit is not set
}
```
**分离（提取）右起第一个为1的位**
```
x & (-x)
```
**分离（提取）右起第一个为0的位**
```
~x & (x+1)
```

**置右起第一个0位为1**
```
x | (x+1)
```

**置右起第一个1位为0**
```
x & (x-1)
```

**n + 1**
```
-~n
```
**n - 1**
```
~-n
```
**符号取反**
```
~n + 1;
(n ^ -1) + 1;
```
**`if (x == a) x = b; if (x == b) x = a;`**
```
x = a ^ b ^ x;
```
**交换相邻位**
```
((n & 10101010) >> 1) | ((n & 01010101) << 1)
```
**m与n最右侧的相反位**
```
(n^m)&-(n^m) // returns 2^x where x is the position of the different bit (0 based)
```
**m与n最右侧的相同位**
```
~(n^m)&(n^m)+1 // returns 2^x where x is the position of the common bit (0 based)
```
## 浮点数

这些是受[fast inverse square root method](https://en.wikipedia.org/wiki/Fast_inverse_square_root)启发而来的技巧。 
大部分为原创。

**float转换为bit数组(unsigned uint32_t)**
```c
#include <stdint.h>
typedef union {float flt; uint32_t bits} lens_t;
uint32_t f2i(float x) {
  return ((lens_t) {.flt = x}).bits;
}
```
<sub>*注：使用union进行转换在C++中是未定义行为，改使用`std::memcpy`。*</sub>

**将bit数组转换回float**
```c
float i2f(uint32_t x) {
  return ((lens_t) {.bits = x}).flt;
}
```

**利用`frexp`从*正*浮点数中近似取出bit数组**

*`frexp`将数值按照2<sup>n</sup>进行分解，即`man, exp = frexp(x)`表示man * 2<sup>exp</sup> = x同时0.5 <= man < 1.*
```c
man, exp = frexp(x);
return (uint32_t)((2 * man + exp + 125) * 0x800000);
```
<sub>*注：此操作最大产生2<sup>-16</sup>相对误差，由于man + 125覆盖了最后8位，保留了尾数的前16位。*</sub>

**快速计算平方根倒数**
```c
return i2f(0x5f3759df - f2i(x) / 2);
```
<sub>*注：此处使用了`i2f`与`f2i`函数。*</sub>

见[Wikipedia](https://en.wikipedia.org/wiki/Fast_inverse_square_root#A_worked_example)。

**借助无穷级数快速计算正数的n次方根**
```c
float root(float x, int n) {
#DEFINE MAN_MASK 0x7fffff
#DEFINE EXP_MASK 0x7f800000
#DEFINE EXP_BIAS 0x3f800000
  uint32_t bits = f2i(x);
  uint32_t man = bits & MAN_MASK;
  uint32_t exp = (bits & EXP_MASK) - EXP_BIAS;
  return i2f((man + man / n) | ((EXP_BIAS + exp / n) & EXP_MASK));
}
```

推导见[此处](http://www.phailed.me/2012/08/somewhat-fast-square-root/)。

**Fast Arbitrary Power**
```c
return i2f((1 - exp) * (0x3f800000 - 0x5c416) + f2i(x) * exp)
```

<sub>*注：`0x5c416`是此方法所给出的偏置值。若带入exp = -0.5，则为快速平方根倒数方法中的常量`0x5f3759df`。*</sub>

此方法推导见[these set of slides](http://www.bullshitmath.lol/FastRoot.slides.html)。

**快速几何平均**

几何平均数是n个数连乘积的n次方根

```c
#include <stddef.h>
float geometric_mean(float* list, size_t length) {
  // Effectively, find the average of map(f2i, list)
  uint32_t accumulator = 0;
  for (size_t i = 0; i < length; i++) {
    accumulator += f2i(list[i]);
  }
  return i2f(accumulator / n);
}
```
推导见[此处](https://github.com/leegao/float-hacks#geometric-mean-1)。

**快速自然对数**

```c
#DEFINE EPSILON 1.1920928955078125e-07
#DEFINE LOG2 0.6931471805599453
return (f2i(x) - (0x3f800000 - 0x66774)) * EPSILON * LOG2
```

<sub>*注：此方法使用`0x66774`作为偏置值。末尾乘以`ln(2)`是因为此方法其余部分计算的为`log2(x)`。*</sub>

推导见[此处](https://github.com/leegao/float-hacks#log-1)。

**快速自然指数**

```c
return i2f(0x3f800000 + (uint32_t)(x * (0x800000 + 0x38aa22)))
```

<sub>*注：偏置值`0x38aa22`对应为基数的乘法系数。特别地，它对应着2<sup>z</sup> = e中的`z`*</sub>

推导见[此处](https://github.com/leegao/float-hacks#exp-1)。

## 字符串

**转换字母为小写**
```
按位或空格字符 => (x | ' ')
结果恒为小写字母，即使原字符已为小写
例如：('a' | ' ') => 'a' ; ('A' | ' ') => 'a'
```

**转换字母为大写**
```
按位与下划线字符 => (x & '_')
结果恒为大写字母，即使原字符已为大写
例如：('a' & '_') => 'A' ; ('A' & '_') => 'A'
```
**字母大小写互换**
```
按位异或空格字符 => (x ^ ' ')
例如：('a' ^ ' ') => 'A' ; ('A' ^ ' ') => 'a'
```
**字母表序号**
```
按位与chr(31)/binary('11111')/(hex('1F') => (x & "\x1F")
结果在1~26之间，大小写无关
例如：('a' & "\x1F") => 1 ; ('B' & "\x1F") => 2
```
**字母表序号（仅大写）**
```
按位与问号字符 => (x & '?') 或按位异或@字符 => (x ^ '@')
例如：('C' & '?') => 3 ; ('Z' ^ '@') => 26
```
**字母表序号（仅小写）**
```
按位异或反引号字符/chr(96)/binary('1100000')/hex('60') => (x ^ '`')
例如：('d' ^ '`') => 4 ; ('x' ^ '`') => 24
```

## 杂项

**使用位移运算快速转换R5G5B5颜色格式至R8G8B8**
```
R8 = (R5 << 3) | (R5 >> 2)
G8 = (R5 << 3) | (R5 >> 2)
B8 = (R5 << 3) | (R5 >> 2)
```
注：使用任何非英文字符会产生错误结果

## 相关资源

* [Bit Twiddling Hacks](https://graphics.stanford.edu/~seander/bithacks.html)
* [Floating Point Hacks](https://github.com/leegao/float-hacks)
* [Hacker's Delight](http://www.hackersdelight.org/)
* [The Bit Twiddler](http://bits.stephan-brumme.com/)
