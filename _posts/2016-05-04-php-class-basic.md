---
title: "【PHP拾遗】系列-类(class)-基础"
category: [tech]
tag: [php]
---


在OOP中类成为了编程中的支柱角色,以前使用PHP的时候都使用了很多,但是一直都没有很好地归纳总结一下,现在有机会重温一下要点,发现有很多很好的特性都没有使用到,甚至有一些概念被一直误解了,所以学习的时候也做一下笔记,方便日后翻阅.

##定义类

以下的类定义会包含本文后面会叙述的一些要点和关键词

```php
<?php
class SimpleClass
{
    // 定义类常量
    const CONSTANT = 'CONSTANT';

    // 定义一个有默认值的属性
    public $default = 'have a default value';

    // 定义不同可见性的属性
    public $public = 'public';
    protected $protected = 'protected';
    private $private = 'private';

    // 定义无默认值的属性
    public $var;

    // 定义静态属性
    public static $static = 'static';

    // 定义类方法
    public function showPrivate()
    {
        // 通过伪变量$this代指主叫对象
        echo $this->private;
    } 

    public function multiShow()
    {
        // 几种不同的访问属性方式
        echo 'this '.$this->public."\n";
        echo 'self '.self::CONSTANT."\n";
        echo 'static '.static::$static."\n";
    }

    // 构造函数
    function __construct()
    {
        echo __CLASS__." __construct init\n";
        $this->var = 'var';
    }

    // 析构函数
    function __destruct()
    {
        echo "__destruct clean up\n";
    }
}
```

## 创建对象

我们有三种方法来创建新的对象

```php
<?php
class SimpleClass
{
    public function getNew()
    {
        return new static;
    }
}
 
// 通过类名创建新对象
$a = new SimpleClass();
 
// 通过已存在的对象创建新的对象
$b = new $a;
 
// 通过static后期绑定得到的类名创建新的对象,关于后期绑定后面的章节会详细叙述
$c = $b->getNew();
```

##静态与非静态

###静态

在类中通过关键词 ```static``` 修饰后的属性或方法就是静态的,静态的属性或方法有几个关键的要点:

* 可以不实例化类就直接访问
* 静态的属性不能通过已经实例化的对象来访问
* 静态的方法可以通过已经实例化的对象来访问,也可以通过类名直接访问
* 静态的方法中代指主叫对象的伪变量 ```$this``` 不可用

比如:

```php
<?php
class SimpleClass
{
    public $non_static = 'non_static';
    public static $static = 'static';
 
    public static function displayStatic()
    {
        echo self::$static;
    }
}
 
$a = new SimpleClass();
 
// 通过实例化的对象访问静态方法
$a->displayStatic();  // output: static
 
// 直接通过类名访问静态方法
SimpleClass::displayStatic();  // output: static
 
// 访问非静态属性
echo $a->non_static;  // output: static
 
// 通过实例化的对象访问静态属性
echo $a->static;  // output: Notice: Accessing static property SimpleClass::$static as non static
```

###非静态

非静态的属性或方法,就是指不经过 ```static``` 修饰的属性或方法,这些属性或方法:

* 可以经过对象来访问
* 方法中可以使用 ```$this``` 来代指主叫对象

##范围解析操作符(::),self,parent和后期静态绑定

为什么把这些放在一起呢,因为我觉得这几个概念的关系比较大

###范围解析操作符(::)

这一对双冒号可以用来访问类中的静态成员,类常量和被覆盖的属性和方法

比如:

```php
<?php
class A
{
    public static $static = 'static';
 
    public static function getIntro()
    {
        echo 'i am A';
    }
}
 
class B extends A
{
    const CONSTANT = 'CONSTANT';
 
    public static function getIntro()
    {
        // 访问被覆盖的方法
        parent::getIntro();
        echo ',but i am B now.';
    }
 
}
 
// 访问静态方法
A::getIntro();  // i am A
// 访问静态属性
echo A::$static;  // output: static
 
B::getIntro();  // output: i am A,but i am B now.
// 访问类常量
echo B::CONSTANT;  // CONSTANT
```

###访问的定位

在上述例子中我们可以清楚看到范围解析操作符(::)的作用,但是依然有一个问题,我们怎么表述自己和父类这个关系?

这个时候我们就可以在类的内部使用关键词 ```self``` 和 ```parent``` 来区分自己和父类.

这个访问方式让我联想到C++中对类的定义的过程,C++中在类的定义的时候会对静态成员和函数预先开辟一个空间来存放,而一些非静态的成员在调用的时候才会动态分配空间,这样静态成员对于同一个类或函数都是同一个内存空间,而PHP也类似这样,所以如果静态成员被子类覆盖的时候,就要通过一些途径来区分好父类子类的关系,让程序员有路线可以寻回变量, ```self``` 和 ```parent``` 就是有这样的用途.

```php
<?php
class A
{
    public static $static = 'A static';
 
    public function showIntro()
    {
        echo 'i am A';
    }
}
 
class B extends A
{
    // 覆盖父类的静态变量
    public static $static = 'B static';
 
    // 覆盖父类方法
    public function showIntro()
    {
        echo 'i am B';
    }
 
    public function showStatic()
    {
        // 找到父类的静态变量
        echo parent::$static;
        echo ',';
        // 自己的静态变量
        echo self::$static;
 
 
        echo "\n";
 
        // 找到父类的方法
        echo parent::showIntro();
        echo ',';
        // 找到自己的方法
        echo self::showIntro();
    }
}
 
$tmp = new B();
 
$tmp->showStatic();  // output: A static,B static  i am A,i am B
```

###后期静态绑定

这个概念的产生是由于**self:: 的限制**: 使用 ```self::``` 或者 ```__CLASS__``` 对当前类的静态引用，取决于定义当前方法所在的类
比如:

```php
<?php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        self::who();
    }
}
 
class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}
 
// 由于test方法在A中定义,所以当使用self::的时候获取到的是A的作用域
 
B::test();  // output: A
 
$b = new B();
$b->test();  // output: A
```

我们可以通过关键字 ```static``` (所以这个关键字有两个功能,静态成员的声明和后期静态绑定)来打破这个限制,

后期绑定 ```static::``` 不再被解析为定义当前方法所在的类,而是代指实际运行时候的类.

```php
<?php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        static::who();
    }
}
 
class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}
 
// 使用static指代调用方法的类,这就叫转发调用 

B::test();  // output: B
 
$b = new B();
$b->test();  // output: B
```

* **非静态环境下使用后期静态绑定**

在非静态环境下，所调用的类即为该对象实例所属的类。由于 $this-> 会在同一作用范围内尝试调用私有方法，而 static:: 则可能给出不同结果。另一个区别是 static:: 只能用于静态属性。

```php
<?php
class A {
    private function foo() {
        echo "success!\n";
    }
    public function test() {
        $this->foo();
        static::foo();
    }
}
 
class B extends A {
   // foo()会复制到B,因此使用的依然是A的作用域,所以调用成功
}
 
class C extends A {
    private function foo() {
        // 原方法被覆盖,新的方法的作用域为C
    }
}
 
$b = new B();
$b->test();  // output: success! success! success!
$c = new C();
$c->test();  // output: Fatal error: Call to private method C::foo() from context 'A'
```

## 类常量

类常量是类中一旦定义后就保持不变的常量,在定义和使用的时候不需要增加 ```$``` 符号
比如:

```php
<?php
class A {
    const CONSTANT = 'CONSTANT';
}
 
// 通过静态方法读取
echo A::CONSTANT;  // output: CONSTANT
 
// 不能够改变
// A::CONSTANT = 'NEW　CONSTANT';  // output: Parse error: syntax error, unexpected '='
```

## 属性

关于类的属性有几个需要注意的要点:

* 属性必须定义可见性
* 方法的可见性默认为public
* 同类的不同对象可以访问相互的protected和private成员

### 属性的可见性

|     /     | Parent | Self | Child | Foreigner |
|:---------:|:------:|:----:|:-----:|:---------:|
|   public  |    O   |   O  |   O   |     O     |
| protected |    O   |   O  |   O   |     X     |
|  private  |    X   |   O  |   X   |     X     |

### 访问属性

在类的内部,不同的属性可以有不同的访问方式

* **非静态属性的访问**

可以通过 ```$this``` 伪变量来指代主叫对象来访问属性

什么是主叫对象呢?
比如:

```php
<?php
class A {
    public $intro = 'i am A';
 
    public function showIntro()
    {
        // 通过$this来访问非静态变量
        echo $this->intro;
    }
}
 
$a = new A();
 
// 这个时候对于showIntro方法中的$this来说$a就是主叫对象
$a->showIntro();
```

* **静态属性的访问**

就像上文讲到的我们可以通过双冒号 ```::``` 加上 ```self``` 或 ```parent``` 来访问相应的静态属性