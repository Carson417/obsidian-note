# 1 概念
## 1.1 说一下Java的特点
- <font color="#0070c0">无关平台</font>：Java编译器将源代码编译成字节码。字节码可在任何安装了JVM的系统上运行
- <font color="#0070c0">面向对象</font>：面向对象的编程语言，易于维护和重用
- <font color="#0070c0">内存管理</font>：Java有自己的垃圾回收机制，自动管理内存和不再使用的对象


## 1.2 Java的优势和劣势
- **优势**：
	- 跨平台
	- 面向对象
	- 生态系统
	- 内存管理
	- 多线程支持
	- 安全性（Java有安全模型，如沙箱机制，适合网络环境）
	- 稳定性
- **劣势**：
	- 性能（相比C++原生编译语言）
	- 语法繁琐
	- 内存消耗（JVM本身占内存）
	- 面向对象过于严格（虽然引入了函数式编程）
	- 开发效率（相比动态语言Python）


## 1.3 Java为什么能跨平台
主要依赖JVM 

<u>JVM也是一个软件，不同平台有不同的版本</u>。Java源码编译后会生成字节码(.class)文件。**JVM负责将字节码文件翻译成特定平台下的机器语言，然后运行**。（即使将Java程序打包成可执行文件），仍然需要JVM支持）

> 不同平台下编译生成的字节码是一样的，但是由JVM翻译成的机器码不一样

<u>跨平台的是Java程序，不是JVM</u>。JVM使用C/C++开发的，是编译后的机器码，不能跨平台，不同平台需要安装不同版本的JVM

![[Pasted image 20260126153709.png|350]]


## 1.4 JVM是什么
java虚拟机，负责解释自己的指令集（字节码）并映射到本地的CPU指令集和OS的系统调用

JVM屏蔽了与OS平台相关的信息，使得java程序只需生成在JVM上运行的目标代码，就可以在多种平台上不加修改的运行


## 1.5 JVM、JRE、JDK 三者关系
![[Pasted image 20260126153907.png|400]]

- JVM是Java虚拟机，是Java的运行环境
	- 负责将字节码解释或编译成机器码，并执行程序。JVM提供内存管理、垃圾回收、安全性等功能
- JRE是Java运行时环境，是Java程序运行所需的最小环境
	- 包括JVM、核心Java类库，用于支持Java程序运行。JRE不包括开发工具，只提供所需的运行环境
- JDK是Java开发工具包，是开发Java程序所需的工具集合
	- 包括JVM、编译器(javac)、调试器(jdb)、类库(Java标准库和开发工具库)。提供了开发、编译、调试、运行Java程序所需的全部工具和环境

>如果开发Java就要装JDK；如果只是运行别人写好的程序只需装JRE 


## 1.6 JVM和Java有什么区别
Java是语言，JVM是平台。一个是写代码的工具，一个是跑代码的环境

![[Pasted image 20260126160013.png|350]]

**JVM不只能跑Java**。JVM执行的是字节码，像Kotlin、Scala等语言，编译后也是是生成Java字节码，一样可以在JVM上运行


## 1.7 编译型语言和解释型语言的区别
- 编译型语言：在程序执行**前**，整个源代码会被编译成机器码或者字节码，生成可执行文件。执行时直接运行编译后的代码，速度快，但跨平台性较差
	- C/C++
- 解释性语言：在程序执行**时**，逐行解释源代码，不生成独立的可执行文件。通常由解释器动态解释并执行代码，跨平台性好，但执行速度慢
	- py、js


## 1.8 Java为什么解释和编译都有
Java既是编译型也是解释型语言，默认采用的混合模式

![[Pasted image 20260126160507.png]]

<font color="#0070c0">编译性：</font>
- 源代码首先被编译成字节码，JIT会把编译过的机器码保存起来，以备下次再用
<font color="#0070c0">解释性：</font>
- JVM中的一个方法调用计数器，当累计计数大于一定值的时候，就使用JIT进行编译生成机器码文件。否则就使用解释器进行解释执行，字节码也是经过解释器进行解释运行的

完整执行流程
```text
.java 源代码
   ↓（javac 编译）
.class 字节码
   ↓（JVM 执行）
   ├─ 解释执行（Interpreter）
   └─ 即时编译（JIT → 本地机器码）
```

JVM运行时主要做了2件事
1. 解释执行
刚启动时，字节码一行一行解释、执行
2. 即时编译（JIT，Just-In-Time Compiler）
当JVM发现热点代码，会将其编译成本地机器码，直接由CPU执行


## 1.9 Python和Java区别
- java是一种已编译的编程语言，java编译器将源代码编译为字节码，JVM执行字节码
- python是一种解释语言，翻译时会在执行程序的同时进行翻译


## 1.10 值传递和引用传递的区别
java只有**值传递**

- **值传递**：传递的是实际值的**副本**，适用基本数据类型，**不会影响原变量的值**
- **引用传递**（本质仍是值传递）：对于引用类型，传递的是**对象引用的副本**（不是对象本身）
	- 两个引用（原引用和副本）指向同一个对象，因此
		- 通过副本修改对象内部数据，会影响原对象
		- 修改副本的指向（如重新赋值），不会影响原引用的指向


---


# 2 数据类型
## 2.1 八种基本数据类型
Java数据类型分为：基本数据类型、引用数据类型

![[Pasted image 20260126171744.png]]

| 数据类型    | 占用字节 | 描述         |
| ------- | ---- | ---------- |
| byte    | 1    | 最小的整数类型    |
| short   | 2    |            |
| int     | 4    | 默认整数类型     |
| long    | 8    | 定义时数值后加L或l |
| float   | 4    | 定义时数值后加F或f |
| double  | 8    | 默认小数类型     |
| char    | 2    | Unicode编码  |
| boolean | 理论是1 |            |

- 字节数
	- 1字节：byte、boolean
	- 2字节：short、char
	- 4字节：int、float
	- 8字节：long、double
- 包装类：Character、Integer、其余都是首字母大写
- <u>char类型无符号，不能为负，从0开始</u>


## 2.2 int和long是多少字节的
- int：4字节，有符号整数
- long：8字节，有符号整数


## 2.3 int和long可以相互转换吗
int转long安全，long转int可能会数据丢失或溢出

将int转换为long可以通过直接赋值或强制类型转换
```java
int value = 10;
long longValue = value;   //自动转换
```

大转小则需要强制转换
![[Pasted image 20260126180201.png]]

- **byte -> char 不能自动提升，只能强转**，因为byte可能是负的，但是char没负的
- **char -> short 不能自动提升，只能强转**，因为虽然都是2字节，但是char没负的


## 2.4 数据类型转换的方式有哪些
- <font color="#0070c0">自动类型转换</font>：当目标类型的范围大于源类型，java会自动转换源类型为目标类型
- <font color="#0070c0">强制类型转换</font>：当目标类型的范围小于源类型，需要强转，可能导致数据丢失溢出
- <font color="#0070c0">字符串转换</font>：Java提供了将字符串表示的数据转换为其他类型数据的方法
- <font color="#0070c0">数值之间的转换</font>：Java提供了一些方法，这些转换可通过包装类来实现


## 2.5 类型转换会出现什么问题吗
基本数据类型转换不说了，说一下对象引用转换的问题

- 向上转型是自动进行的，而且安全
```java
Class Dog extends Animal{}
Dog dog = new Dog();
Animal animal = dog;
```

- 向下转型需要手动进行，并有风险
	- 如果父类对象实际不是目标子类的实例，在转型时就会抛错
```java
Animal animal = new Animal();
Dog dog = (Dog) animal;   // 运行时抛出ClassCastException
```

原因是Java对象在运行时会记录其真实类型，当向下转型，java会检查对象的实际类型是否与目标类型兼容。不兼容就会抛出异常
**看真实对象（看右边 new 的是谁，不看左边写的是什么）是不是目标类型或其子类**

解决办法：使用`instance`检查
```java
if(animal instance Dog){
	Dog dog = (Dog) animal;
}
```

**instanceof 后面的类型 必须是 真实对象类型的父类 / 本类 / 接口**


## 2.6 为什么用BigDecimal不用Double
double会出现精度丢失问题，二进制有时不能精确的表示一个小数
decimal是精确计算，可以确保精确的是十进制数计算

```java
BigDecimal num1 = new BigDecimal("0.1");
BigDecimal num2 = new BigDecimal("0.2");

BigDecimal sum = num1.add(num2);
```

注意，**要使用字符串作为参数**，而不是浮点数，避免精度丢失

bigdecimal本质：$value = unscaledValue × 10^{-scale}$

 
## 2.7 装箱和拆箱
基本数据类型和其对应包装类之间的转换

自动装箱主要发生在两种情况：<font color="#0070c0">赋值、方法调用</font>
- 赋值时，现在所有的转换都是由编译器完成的
```java
Integer ob = 3;   // 自动装箱
int i = ob;   // 自动拆箱
```
- 方法调用时，可以传入原始数据值或者对象，编译器会帮我们转换

自动装箱的弊端：**循环时可能会创建多余对象**，下面的例子，sum会自动拆箱，相加后再装箱
```java
Integer sum =0;
for(int i = 0; i < 500; i++){
	sum += i;
}

// 实际是
int result = sum.intValue() + i;
Integer sum = new Integer(result);
```


## 2.8 Java为什么要有Integer
- 泛型中的应用，**泛型只能使用引用类型**
- 转换中的应用，基本类型和引用类型不能直接进行转换，如int不能直接变成String
- 集合中的应用，集合中只能存储对象，不能存储基本数据类型


## 2.9 Integer相比int有什么优点
区别：
- 基本类型和引用类型：基本数据类型是java中最基本的数据类型，它们是预定义的，不用实例化就可以使用。而引用类型则需要通过实例化对象使用
	- 用int存储整数时，不需要额外的内存分配；用Integer，需要为对象分配内存
- 自动装箱和拆箱
- 空指针：对一个未经初始化的Integer变量进行操作，会出现空指针异常；<u>null无法进行自动拆箱</u>


## 2.10 为什么要保留int类型
在读写、存储效率上，基本数据类型更高效
- 包装类是引用类型，对象的引用和对象本身是分开存储的；基本类型数据，变量对应的内存块直接存储数据本身
- 在开启引用压缩的JVM上，Integer占16字节；int占4字节


## 2.11 Integer缓存
**Integer内部实现了静态缓存池，用于存储特定范围内的整数对应的Integer对象**
默认-128至127，当用Integer.valueOf(int）创建一个这个范围内的整数对象时，并不会每次都生成新的对象实例，而是复用缓存中现有的对象，直接从内存中取出


---


# 3 面向对象
## 3.1 封装继承多态
- <font color="#0070c0">封装</font>：将对象的属性和行为结合在一起，对外隐藏对象的内部细节，仅通过对象提供的接口与外界交互
- <font color="#0070c0">继承</font>：继承是一种可以使得子类自动共享父类数据结构和方法的机制
- <font color="#0070c0">多态</font>：多态指允许不同类的对象对同一消息作出响应。即同一个接口，使用不同的实例而执行不同操作。多态分编译时多态（重载）和运行时多态（重写）


## 3.2 多态体现在哪几个方面
- <font color="#0070c0">方法重载</font>：同一个类中可以有多个同名方法，它们具有不同的参数列表（**参数类型、数量或顺序不同**）。<u>编译器会在编译时确定调用哪个方法</u>
- <font color="#0070c0">方法重写</font>：子类能提供对父类中同名方法的具体实现。在<u>运行</u>时JVM会根据对象的实际类型确定调用哪个版本的方法
- <font color="#0070c0">接口与实现</font>：多个类可以实现同一个接口，并且**用接口类型的引用来调用这些类的方法**
- <font color="#0070c0">向上转型和向下转型</font>：
	- 向上转型：子类对象赋值给父类/接口引用（通常自动完成）。引用只能访问父类/接口声明的方法，但若方法被重写，运行时会执行子类版本（动态绑定）
	- 向下转型：将父类/接口引用强制转换为某个子类类型，以访问子类特有方法。转换前需保证引用的真实对象确实是该子类（或其子类），通常用 `instanceof` 判断，否则会抛错

第三点举例：
```java
// 接口
interface Payment{
	void pay(int amount);
}
// 实现1
class WechatPay implements Payment{
	public void pay(int amount){}
}
// 实现2
class AliPay implements Payment{
	public void pay(int amount){}
}

// 接口类型引用
Payment p1 = new WechatPay();
Payment p2 = new AliPay();
p1.pay(100);
p2.pay(100);
```

第四点举例（*注意注释*）：
```java
// 向上转型
Animal a = new Dog();
Payment p = new AliPay(); // 接口也一样
// 使用
a.eat();   // Animal 有就能调用 ，注意执行时是Dog重写后的方法
a.bark();   // Dog才有 -> 编译不通过


// 向下转型
Animal a = new Dog();
Dog d = (Dog) a;

// 为什么要用instance
Animal a = new Cat();
Dog d = (Dog) a;   // 报错
if(a instanceOf Dog){
	Dog d = (Dog) a; 
}
```


## 3.3 面向对象的设计原则
六大设计原则：
- <font color="#0070c0">单一职责原则（SRP）</font>：一个类应该只有一个引起它变化的原因，即一个类应该只负责一项职责
- <font color="#0070c0">开放封闭原则（OCP）</font>：软件实体应该对扩展开放，对修改封闭
- <font color="#0070c0">里氏替换原则（LSP）</font>：子类对象应该能够替换掉所有父类对象
- <font color="#0070c0">接口隔离原则（ISP）</font>：客户端不应该依赖那些它不需要的接口
- <font color="#0070c0">依赖倒置原则（DIP）</font>：高层模块不应该依赖底层模块，二者都应该依赖抽象
- <font color="#0070c0">最少知识原则（Law of Demeter）</font>：一个对象应当对其他对象有最少的了解


## 3.4 重写与重载有什么区别
- 重载指的是在同一个类中，可以有多个同名方法，它们具有不同的参数列表（参数类型、个数、顺序），编译器根据调用时的参数类型决定调用哪个方法
- 重写指的是子类可以重新定义父类中的方法，但是注意方法名、参数列表、返回类型必须和父类中的方法一致


## 3.5 抽象类和普通类的区别
- 实例化：抽象类只能被继承，不能直接实例化
- 方法实现：抽象类中方法可以有实现，也可以没实现


## 3.6 抽象类和接口的区别
- 两者的特点：
	- 抽象类用于描述类的共同特性和行为。适用与有明显继承关系的场景
	- 接口用于定义行为规范，有常量、抽象方法、默认方法、静态方法
- 两者的区别：
	- 实现方式：implements、extends
	- 方法方式：接口只能有定义，不能有方法的实现，java8中可以定义default方法体；抽象类可以有实现与定义
	- 访问修饰符：
		- 接口
			- 成员变量默认`public static final`，必须赋初值，不能修改
			- 成员方法：`public`、`abstract`
		- 抽象类
			- 成员变量默认`default`，可在子类中重定义，也可重新赋值
			- 抽象方法：被`abstract`修饰，<u>不能被private、static、synchronized、native修饰</u>，以分号结尾，不带花括号
	- 变量：抽象类可包含实例变量和静态变量，**接口只能包含静态常量**


## 3.7 抽象类能加final修饰吗 
不能，抽象类是用来被继承的，**final修饰符用于禁止类被继承或方法被重写**


## 3.8 接口里可以定义哪些方法
- <font color="#0070c0">抽象方法</font>
	- 所有实现接口的类都必须实现这些方法
	- 抽象方法默认是`public`和`abstract`，修饰符可省略
- <font color="#0070c0">默认方法</font>
	- 允许接口提供具体实现
	- 实现类可以重写默认方法
```java
public interface Animal{
	void makesound();
	
	default void sleep(){
		...
	}
}
```
- <font color="#0070c0">静态方法</font>
	- 属于接口本身，可直接通过接口名调用
```java
public interface Animal{
	void makesound();
	
	static void sleep(){
		...
	}
}
```
- <font color="#0070c0">私有方法</font>
	- 用于在接口中为默认方法或其他私有方法提供辅助功能
	- 这些方法不能被访问类实现，只能在接口内使用
```java
public interface Animal{
	void makesound();
	
	default void sleep(){
		logSleep();
	}
	
	private void logSleep(){...}
}
```


## 3.9 抽象类可以被实例化吗
不能
抽象类可以有构造器，这些构造器在子类实例化时会被调用，这个过程是为了创建子类的实例


## 3.10 接口可以包含构造函数吗
不可以，因为接口不会有自己的实例
> 构造函数是初始化class的属性或方法，在new的瞬间调用


## 3.11 静态变量和静态方法
静态，与类有关，不与类的实例对象关联。<u>它们在内存中只存在一份，可以被类的所有实例共享</u>

- 静态变量，static修饰，属于类而不属于具体对象，一般用作计数器、常量
	- 共享性：所有实例共享一个静态变量。一个实例修改其值，其他都能看到
	- 初始化：**静态变量在类被加载时初始化，只会对其进行一次分配内存**
	- 访问方式：可通过类名或实例访问

- 静态方法，类似静态变量，一般用作助手方法
	- 无实例依赖：可在没有创建类实例的情况下调用。**静态方法不能直接访问非静态成员变量或方法**
	- 访问静态成员：静态方法不能直接访问非静态成员
	- 多态性：**静态方法不支持重写，但可被隐藏**


## 3.12 非静态内部类和静态内部类的区别
- <u>前依赖外部类的实例，后不依赖</u>
- <u>前可以访问外部类的实例变量和方法，后只能访问外部类的静态成员</u>
- <u>前内部不能定义静态成员，后可以</u>
- <u>前在外部实例化后才能实例化，后可以独立实例化</u>
- <u>前可以访问外部类的私有成员，后不能直接访问外部类的私有成员，需要通过实例化外部类来访问</u>


## 3.13 非静态内部类可以直接访问外部方法，编译器是怎么做到的
**因为编译器在生成字节码时会为非静态内部类维护一个指向外部类实例的引用**


---


# 4 关键字
## 4.1 final的作用
- <font color="#0070c0">修饰类</font>：这个**类不能被继承**（String就是）
- <font color="#0070c0">修饰方法</font>：这个**方法不能在子类中被重写**（Object的getClass()）
- <font color="#0070c0">修饰变量</font>：**基本数据类型，一旦被赋值就不能再改变；引用类型，不能再指向其他对象，但对象本身的内容可以变**


## 4.2 static的作用
将成员与类关联，而非与类的实例关联

- <font color="#0070c0">修饰变量</font>：被修饰的变量属于类本身，而非类的实例。所有对象共享一份静态变量，内存中只存在一份副本。可以通过类名或实例访问。通常用于计数器、常量
- <font color="#0070c0">修饰方法</font>：属于类，不属于实例，因此**不能直接访问类中的非静态成员**（因为非静态成员依赖对象存在），适用于工具类方法
- <font color="#0070c0">修饰代码块</font>：**静态代码块再类加载时执行，且只执行一次**（优于对象构造方法），用于初始化静态变量或执行类级别的预处理。<u>多个静态代码块按定义顺序执行，且先于非静态代码块和构造方法</u>
- <font color="#0070c0">修饰内部类</font>：静态内部类不依赖于外部类的实例，可以独立存在，不能直接访问外部类的非静态成员（需通过外部类实例访问）


---


# 5 深拷贝和浅拷贝
## 5.1 深拷贝和浅拷贝的区别
![[Pasted image 20260128170940.png|500]]

- 浅拷贝：浅拷贝会创建一个新的对象，并复制原对象中所有字段的值。 
	- 对于基本类型字段，复制的是具体数值  
	- 对于引用类型字段，复制的是引用地址，新旧对象会指向同一个被引用对象
- 深拷贝：在复制对象的同时，将对象内部的所有引用类型字段内容也复制一份，而不是共享引用


## 5.2 实现深拷贝的三种方法
- <font color="#0070c0">实现 Cloneable 接口并重写 clone() 方法</font>：要求对象及其所用引用类型字段都实现 Cloneable接口，并且重写clone() 方法。在方法中，通过递归克隆引用类型字段来实现深拷贝
```java
class MyClass implements Cloneable{
	private String field1;
	private NestedClass nestedObject;
	
	@Override
	protected Object clone() throws CloneNotSupportedException{
	    // Object的clone
		MyClass cloned = (MyClass) super.clone(); 
		// 深拷贝内部的引用对象
		cloned.nestedObject = (NestedClass) nestedObject.clone();
		return cloned;
	}	

	class NestedClass implements Cloneable{
		private int nestedField;
		
		@Override
		protected Object clone() throws CloneNotSupportedException{
			return super.clone();
		}
	}
}
```

- <font color="#0070c0">使用序列化和反序列化</font>：通过将对象序列化为字节流，再从字节流反序列化为对象来实现深拷贝。要求对象及其所有引用字段都实现Serializable接口
```java
class MyClass implements Serializable{
	private String field1;
	private NestedClass nestedObject;
	
	public MyClass deepCopy(){
		try{
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream oos = new ObjectOutputStream(bos);
			oos.writeObject(this);
			oos.flush();
			oos.close();
		
			ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
			ObjectInputStream ois = new ObjectInputStream(bis);
			return (MyClass) ois.readObject();
		} catch(...){...}
	}

	class NestedClass implements Serializable{
		private int nestedField;
	}
}
```

- <font color="#0070c0">手动递归复制</font>：针对特定对象结构，手动递归复制对象及其引用类型字段
```java
class MyClass implements Cloneable{
	private String field1;
	private NestedClass nestedObject;
	
	public MyClass deepCopy(){
		MyClass copy = new MyClass();
		copy.setField1(this.field1);
		copy.setNestedObject(this.nestedObject.deepCopy());
		return copy;
	}
}


class NestedClass{
	private int nestedField;
	
	public NestedClass deepCopy(){
		NestedClass copy = new NestedClass();
		copy.setNestedField(this.nestedField);
		return copy;
	}
}
```


---


# 6 泛型
## 6.1 什么是泛型
泛型允许类、接口和方法在定义时使用一个或多个类型参数，这些参数在使用时可以被指定为具体的类型
主要目的：在编译时提供更强的类型检查，并在编译后能保留类型信息，避免在运行时出现类型转换异常

- 如果没有泛型会发生什么
    ```java
    List list = new ArrayList();

	list.add("hello");
	list.add(100);   
	list.add(true);  
    ```
取出来时：
```java
String s = (String) list.get(0); // 必须强转
```
有两个问题：
```java
// Q1：编译期不安全
list.add(100);
String s = (String) list.get(1); // 运行时 ClassCastException

// Q2：代码全是强制类型转换
(String)
(Integer)
(User)
```

- 泛型是为了解决什么
**把类型错误提前到编译器发现**

- 什么是泛型
泛型就是把数据类型当成参数传给类或方法，告诉编译器：这个容器里，只允许放某一种类型
```java
List<String>
List<Integer>
Map<String, User>
```

- 经典实例
```java
List<String> list = new ArrayList<>();
list.add("hello");   // list.add(100); ❌ 编译直接报错
String s = list.get(0); // 不需要强转
```

- 泛型核心的作用
（1）类型安全：**错误在编译期暴露**
```java
List<String> list = new ArrayList<>();
list.add(123); // 编译错误
```
（2）消除强制类型转化
```java
String s = list.get(0);
String s = (String) list.get(0);
```
（3）代码复用
```java
Box<T>
Pair<K, V>
```

- 泛型在java中是怎么实现的
java泛型是伪泛型：**编译期泛型，运行期擦除**（类型擦除）
```java
// 编译前
List<String>
List<Integer>

// 编译后（字节码）
List
List

// 所以
List<String> list1 = new ArrayList<>();
List<Integer> list2 = new ArrayList<>();
// true
System.out.println(list1.getClass() == list2.getClass());
```

- 泛型有很多限制
（1）不能 `new T()`，运行期T被擦除成Object（或上界）,JVM不知道该new什么
（2）不能 `T.class` / `instance T`，运行期没有T的真实类型信息
（3）不能创建泛型数组`new T[10]` / `new List<String>[10]`，数组是运行期带类型信息的，数组在运行期会检查类型；泛型是编译期的，运行期会擦除
（4）不能在`static`成员里使用类的类型参数T，static属于类，不属于对象，而T是实例化这个类时才确定的
```java
class Test<T> {
    static T value; // ❌
}
// Foo.value到底是String还是Integer
Foo<String> f1;
Foo<Integer> f2;
```
（5）不能重载“只因泛型不同”的方法（签名擦除冲突），擦除后方法签名一样：print(List)，JVM 无法区分

- 泛型边界
```java
<T extends Number>
<? super Integer>
```


## 6.2 为什么需要泛型
- <font color="#0070c0">使用多种数据类型执行相同代码</font>
```java
public static <T extends Number> double add(T a, T b){
	return a.doubleValue() + b.doubleValue();
}
```

- <font color="#0070c0">泛型中的类型在使用时指定，不需要强转</font>（类型安全，编译器会检查）
```java
// 里面的元素是Object，取出需要强转
List list = new ArrayList();
// 无需
List<String> list = new ArrayList<String>();
```


---


# 7 对象
## 7.1 java创建对象有哪些方式
- <font color="#0070c0">new</font>：调用类的构造器来实例化对象
- <font color="#0070c0">Constructor类的newInstance()方法</font>：反射机制，运行时动态创建对象，不需要再编译时知道具体的类
	- Class类的newInstance()方法过时，因为其只能调用无参公有构造器
```java
Constructor<MyClass> constructer = MyClass.class.getConstructor();
MyClass obj = constructor.newInstance();
```
- <font color="#0070c0">clone()方法</font>：通过实现Cloneable接口并重写Object类的clone()方法，可以基于一个现有对象创建一个新的副本对象
	- 注意，Object.clone()默认是浅拷贝，对于引用类型的字段，复制的是引用地址，而不是引用对象本身。如果要深拷贝，需要在clone()方法中手动对 引用对象进行克隆
- <font color="#0070c0">反序列化</font>：通过ObjectInputStream从一个字节流（通常是文件或网络）中重建一个对象，类需要实现Serializable接口，不会调用任何类的构造器
- <font color="#0070c0">工厂</font>：一种设计模式，不直接用new，通过一个方法返回对象实例
	- Integer.valueOf(int)，Calendar.getInstance()
```java
public class Person{
	private String name;
	
	private Person(String name){
		this.name = name;
	}
	
	// 静态工厂
	public static Person createPerson(String name){
		return new Person(name);
	}
}
```


## 7.2 new出的对象什么时候回收
垃圾回收器(GC) 负责回收。GC的工作是在程序运行过程中自动进行的，会周期性检测不再被引用的对象，并将其回收释放内存

回收时机的参考：
- <font color="#0070c0">引用计数法</font>：某个对象的引用计数为0时，表示对象不再被引用，可以被回收
- <font color="#0070c0">可达性分析算法</font>：从根对象（如方法区中的类静态属性、方法中的局部变量等）出发，通过对象之间的引用链进行遍历，如果存在一条引用链到达某个对象，说明该对象可达，反之不可达，不可达的对象将被回收
- <font color="#0070c0">终结器（Finalize）</font>：如果对象重写了finalize() 方法，GC会在回收该对象之前调用finalize() 方法，对象可在此方法中进行一些清理操作。但不推荐


## 7.3 如何获取私有对象
- <font color="#0070c0">通过公共访问器（getter）</font>：通常会为私有成员变量提供公共访问器方法
- <font color="#0070c0">反射机制</font>：允许在运行时检查和修改类、方法、字段等信息。通过反射可以绕过private访问修饰符的限制来获取私有对象
```java
MyClass obj = new MyClass();
Class<?> clazz = obj.getClass();
Field field = clazz.getDeclaredField("privateField");
field.setAccessible(true);
String value = (String) field.get(obj);
```


---


# 8 反射
## 8.1 什么是反射
反射是在运行状态中，对于任意一个类，都能知道这个类中的所有属性和方法，对于任意一个对象，都能够调用它的任意一个方法和属性

反射有以下特性：
- <font color="#0070c0">运行时类信息访问</font>：反射机制允许程序在运行时获取类的完整结构信息，包括类名、包名、父类、实现的接口、构造函数、方法和字段等
- <font color="#0070c0">动态对象创建</font>：可使用反射API动态创建对象实例，即使在编译时不知道具体的类名。这是通过Class类或Constructor对象的newInstance(）方法实现的
- <font color="#0070c0">动态方法调用</font>：可以运行时动态的调用对象的方法，包括私有方法。通过Method类的invoke()方法，允许传入对象实例和参数值来执行方法
- <font color="#0070c0">访问和修改字段值</font>：反射允许程序在运行时访问和修改对象的字段值。通过Field类的get()和set()方法完成

![[Pasted image 20260130105658.png]]


## 8.2 反射的应用场景
- <font color="#0070c0">加载数据库驱动</font>
项目中数据库可能会用很多种，需要动态地根据实际情况加载驱动类，这是我们在使用JDBC连接数据库时使用Class.forName() 通过反射加载数据库的驱动程序
```java
Class.forName("com.mysql.cj.jdbc.Driver");
```
- <font color="#0070c0">配置文件加载</font>
Spring框架的IOC（动态加载管理Bean），Spring通过配置文件配置各种bean
Spring通过XML配置模式装载bean的过程：
1. 将程序中的所欲XML或properties配置文件加载到内存
2. Java类里面解析xml或properties里面的内容，得到对应实体类的字节码字符串以及相关属性信息
3. 使用反射，根据字符串获得某个类的Class实例
4. 动态配置实例的属性


---


# 9 注解
## 9.1 注解的原理
注解本质是**一个继承了Annotation的接口，其具体实现类是Java运行时生成的动态代理类**

我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池


## 9.2 对注解解析的底层实现
注解本质是一种特殊的接口，继承自Annotation接口，注解也叫声明式接口
```java
public @interface MyAnnotation{
	String value();
}
```
编译后，Java编译器会将其转换为一个继承自Annotation的接口，并生成相应的字节码文件

根据注解的作用范围k可有以下类型：
- <font color="#0070c0">源码级别注解</font>：仅存在源码中，编译后不会保留（`@Rentention(RententionPolicy.SOURCE)`）
- <font color="#0070c0">类文件级别注解</font>：保留在 .class 文件中，但运行时不可见（`@Rentention(RententionPolicy.CLASS)`）
- <font color="#0070c0">运行时注解</font>：保留在 .class 文件中，并且可以通过反射在运行时访问（`@Rentention(RententionPolicy.RUNTIME)`）

只有运行时注解可以通过反射机制进行解析。

当注解被标记为 `RUNTIME` 时，Java编译器会在生成的 .class 文件中保存注解信息。这些信息存储在字节码的属性表中，包括一下内容：
- `RuntimeVisibleAnnotations`：存储运行时可见的注解信息
- `RuntimeInvisibleAnnotations`：存储运行时不可见的注解信息
- `RuntimeVisibleParameterAnnotations` 和 `RuntimeInvisibleParameterAnnotations`：存储方法参数上的注解信息

通过工具（如 javap -v ）可以查看 .class 文件中的注解信息

解析注解的基本流程：
（1）获取注册信息：通过反射API可以获取类、方法、字段等元素上的注解
```java
Class<?> clazz = MyClass.class;
MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);
```
（2）底层原理：反射机制的核心类是：`AnnotatedElement`，他是所有可以被注解修饰的元素(Class、Method、Field)的父接口。该接口提供了以下方法：
- `getAnnotation(Class<T> annotationClass)`：获取指定类型的注解
- `getAnnotations()`：获取所有注解
- `isAnnotationPresent(Class<? extends Annotation> annotationClass)`:判断是否包含指定注解











































































