浮点数的运算可能会丢失精度!
```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```
这个和计算机保存小数的机制有很大关系。而且计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，会导致小数精度发生损失的情况。这也解释了为什么十进制小数没有办法用二进制精确表示

## 1 BigDecimal介绍
`BigDecimal` 可以实现对小数的运算，不会造成精度丢失
> 浮点数之间的等值判断，基本数据类型不能用 == 来比较，包装数据类型不能用 equals 来判断
![[Pasted image 20260122112359.png]]

想要解决浮点数运算精度丢失这个问题，可以直接使用 BigDecimal 来定义小数的值，然后再进行小数的运算操作即可
```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x.compareTo(y));// 0
```

## 2 BigDecimal常见方法
### 2.1 创建
推荐使用它的`BigDecimal(String val)`构造方法或者 `BigDecimal.valueOf(double val)` 静态方法来创建对象

![[Pasted image 20260122113614.png]]

### 2.2 加减乘除
`add` 方法用于将两个 BigDecimal 对象相加，`subtract` 方法用于将两个 BigDecimal 对象相减。`multiply` 方法用于将两个 BigDecimal 对象相乘，`divide` 方法用于将两个 BigDecimal 对象相除

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.add(b));// 1.9
System.out.println(a.subtract(b));// 0.1
System.out.println(a.multiply(b));// 0.90
System.out.println(a.divide(b));// 无法除尽，抛出ArithmeticException异常
System.out.println(a.divide(b, 2, RoundingMode.HALF_UP));// 1.11
```

使用 `divide` 方法的时候尽量使用 3 个参数版本，并且`RoundingMode` 不要选择 `UNNECESSARY`，否则很可能会遇到 `ArithmeticException`（无法除尽出现无限循环小数的时候），其中 `scale` 表示要保留几位小数，`roundingMode` 代表保留规则

```java
public BigDecimal divide(BigDecimal divisor, int scale, RoundingMode roundingMode) {
    return divide(divisor, scale, roundingMode.oldMode);
}
```
保留规则：
```java
public enum RoundingMode {
   // 2.4 -> 3 , 1.6 -> 2
   // -1.6 -> -2 , -2.4 -> -3
   UP(BigDecimal.ROUND_UP),
   // 2.4 -> 2 , 1.6 -> 1
   // -1.6 -> -1 , -2.4 -> -2
   DOWN(BigDecimal.ROUND_DOWN),
   // 2.4 -> 3 , 1.6 -> 2
   // -1.6 -> -1 , -2.4 -> -2
   CEILING(BigDecimal.ROUND_CEILING),
   // 2.5 -> 2 , 1.6 -> 1
   // -1.6 -> -2 , -2.5 -> -3
   FLOOR(BigDecimal.ROUND_FLOOR),
   // 2.4 -> 2 , 1.6 -> 2
   // -1.6 -> -2 , -2.4 -> -2
   HALF_UP(BigDecimal.ROUND_HALF_UP),
   //......
}
```

### 2.3 大小比较
`a.compareTo(b)` : 返回 -1 表示 a 小于 b，0 表示 a 等于 b ， 1 表示 a 大于 b

### 2.4 保留几位小数
通过 `setScale` 方法设置保留几位小数以及保留规则
```java
BigDecimal m = new BigDecimal("1.255433");
BigDecimal n = m.setScale(3,RoundingMode.HALF_DOWN);
System.out.println(n);// 1.255
```


## 3 BigDecimal等值比较问题
	





























































