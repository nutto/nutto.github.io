---
title: "【PHP拾遗】系列-类(class)-进阶"
category: [tech]
tag: [php]
---


在之前的[【PHP拾遗】系列-类(class)-基础](http://blog.nuttopan.cn/blog/php-class-basic/)中介绍了PHP的类的基本概念, 但是PHP的类还有一些高级的特性,充分了解这些特性对于OOP会事半功倍.


## 对象的继承

* 子类会继承父类中公有的和受保护成员
* 子类可以覆盖父类继承下来的成员
* 父类必须在子类声明之前声明

我们举个例子:

```php
<?php
class Foo
{
    public $a = 'Foo::a';
 
    public function showName()
    {
        echo "i am Nutto\n";
    }
 
    public function showIntro()
    {
        echo "i am Foo\n";
    }
}
 
class Child extends Foo
{
    // 覆盖父类成员
    public function showIntro()
    {
        echo "i am Child\n";
    }
}
 
$child = new Child();
 
// 继承了父类的属性
echo $child->a."\n";  // output: Foo::a
// 继承了父类的方法
$child->showName();  // output: i am Nutto
// 覆盖了父类的方法
$child->showIntro();  // output: i am Child
```

## 抽象类

* 抽象类不能被实例化
* 抽象类中至少有一个抽象方法
* 子类必须实现抽象类中的所有方法
* 子类实现父类的抽象方法的时候必须遵循
    1. 可见性必须跟父类一样或更宽松(宽松程度: public > protected > private)
    2. 调用方式必须与父类匹配,即类型和所需的参数数量必须一致(子类必须要兼容父类的所有参数,但也可以扩展带默认值的参数)

举个例子:

```php
<?php
abstract class AbstractClass
{
    // 必须至少有一个抽象方法
    abstract protected function showName();
    abstract protected function showNumber();
    abstract public function showAddress($country, $city);
 
 
    public function showWeather()
    {
        echo "The weather is sunny\n";
    }
}
 
class Foo extends AbstractClass
{
    // 要实现所有的抽象方法
    // 可以使用更宽松的可见性
    public function showName()
    {
        echo "my name is Nutto\n";
    }
    // 可见性至少和父类一样
    protected function showNumber()
    {
        echo "my number is 123456\n";
    }
 
    // 满足父类的参数的同时,可以扩展带默认值的参数
    public function showAddress($country, $city, $dist='')
    {
        echo "i am in {$country}-{$city}-{$dist}";
    }
}
 
 
$a = new Foo();
 
$a->showWeather();  // output: The weather is sunny
$a->showName();  // output: my name is Nutto
$a->showAddress('a', 'b', 'c');  // output: i am in a-b-c
```

## 接口

* 接口用于定义需要实现的会向外暴露的方法,但不需要定义这些方法的具体内容
* 接口中的方法都是公有的
* 一个类可以实现多个接口,每一个接口的实现都必须实现该接口的所有方法
* 接口也可以继承
* 接口可以定义常量,但接口的常量不能被覆盖

举个例子:

```php
<?php
interface bodyTagInterface
{
    // 定义接口常量
    const SELF_NAME = 'body';
    public function showTag();
}
 
interface renderInterface
{
    public function showRender();
}
 
// 接口间可以继承
interface templateInterface extends bodyTagInterface
{
    public function showHtml();
}
 
// Template类实现templateInterface接口和renderInterface接口
class Template implements
    templateInterface,
    renderInterface
{
    // 必须实现接口的所有方法
    public function showTag()
    {
        echo self::SELF_NAME."\n";
    }
 
    public function showHtml()
    {
        echo "html\n";
    }
 
    public function showRender()
    {
        echo "render\n";
    }
}
 
$a = new Template();
 
$a->showTag();  // output: body
$a->showHtml();  // output: html
$a->showRender();  // output: render
```

## Trait(特性)

Trait用于服用代码,类似于多继承但是关系更加水平化而不像继承这样的垂直父子关系.

Trait抛除了接口不能写具体方法实现的限制,更像是对一个对象进行有目的的肢解

* 只能用于与class组合,不能用过自身实例化
* 成员的覆盖优先级为: 子类 > Trait > 父类
* Trait可以使用不同的Trait来组合自身
* 与继承不同的是,如果Trait中有静态成员,组合到不同的类中的时候,这些静态成员在不同的类中有不同的空间.

举个例子:

```php
<?php
trait returnSelfProfile
{
    public function showName()
    {
        echo "my name is Nutto\n";
    }
}
 
trait returnSelfBalance
{
    public function showBalance()
    {
        echo "i have 100rmb\n";
    }
}
 
trait returnSelfInfo
{
    public static $name = 'Undefine';
    // 可以使用不同的Trait组合自身
    use returnSelfProfile, returnSelfBalance;
}
 
 
class Person
{
    // 使用Trait组合对象
    use returnSelfInfo;
}
 
class Animal
{
    use returnSelfInfo;
}
 
$me = new Person();
 
// 不同类的同一个Trait的静态变量有不同的空间
Person::$name = 'me';
Animal::$name = 'dog';
 
echo Person::$name."\n";  // output: me
echo Animal::$name."\n";  // output: dog
 
$me->showName();  // output: my name is Nutto
$me->showBalance();  // output: i have 100rmb
```


## 匿名类

匿名类在PHP中是一个新的事物,到PHP7+才支持匿名类

* 使用匿名类的构造器(class)可以快速创建一个一次性的简单对象
* 类中的匿名类不能直接访问外部类的private和protected方法或属性,为了访问外部类(Outer class)protected 属性或方法,匿名类可以 extend(扩展)此外部类.为了使用外部类(Outer class)的 private 属性,必须通过构造器传进来.

举个例子:

```php
<?php
interface simpleInterface {};
trait simpleTrait {};
class ParentClass {};
 
 
// 用匿名类的构造器创建对象,可以像普通类一样继承,实现接口和使用特性组合
var_dump(new class('Nutto') extends ParentClass implements simpleInterface {
    use simpleTrait;
    private $name;
 
    public function __construct($name)
    {
        echo $name . "\n";  // output: Nutto
        $this->name = $name;
    }
});
 
class Outer
{
    private $private_outer = 'private_outer';
    protected $protected_outer = 'protected_outer';
 
    public function changeVar()
    {
        $this->private_outer = 'changed_private_outer';
        $this->protected_outer = 'changed_protected_outer';
    }
 
    public function printer()
    {
        // 嵌套的匿名类要使用外层类的私有成员要通过构造器传值
        return new class($this->private_outer) extends Outer {
            private $anonymous = 'private_anonymous';
            private $tmp;
 
            public function showInfo()
            {
                // 嵌套的匿名类通过extends外层类,就可以使用外层类
                // 的protected成员,这个和继承是同一个概念
                echo $this->protected_outer . "\n" .
                    $this->tmp . "\n" .
                    $this->anonymous . "\n";
            }
 
            public function __construct($private)
            {
                $this->tmp = $private;
            }
        };
    }
}
 
$a = new Outer();
 
$a->printer()->showInfo();  // output: protected_outer private_outer private_anonymous
 
// 其实嵌套的匿名类使用外层类的成员,与继承和传值的概念是一样的,并没有什么魔术处理过程
$a->changeVar();
$a->printer()->showInfo();  // output: protected_outer changed_private_outer private_anonymous
```

## final关键字

* 父类的方法声明为final,则子类不能覆盖这个方法
* 如果一个类声明为final,则这个类不能被继承

举例尝试覆盖final方法:

```php
<?php
class A
{
    final public function showName()
    {
        echo 'A';
    }
}
 
class B extends A
{
    // 会引发错误: Fatal error:  Cannot override final method A::showName()
    public function showName()
    {
        echo 'B';
    }
}
 
$b = new B();
$b->showName();
```

举例尝试继承final类

```php
<?php
final class A {};
 
// 会引发错误: Fatal error:  Class B may not inherit from final class (A)
class B extends A {};
```

## 重载

PHP的重载概念与大多数语言的重载概念不一样,通常的重载概念指的是:同名类的方法因为参数的不一样可以有不同的实现.

PHP的重载指的是当调用**当前环境下未定义或者不可见(下文统称为*不可访问*)**的类属性或方法的时候,一些魔术方法会被调用.

### 属性重载

属性重载的魔术方法有:

```php
public void __set ( string $name , mixed $value )  // 给不可访问属性赋值时调用
public mixed __get ( string $name )  // 读取不可访问属性的值时调用
public bool __isset ( string $name )  // 当对不可访问属性调用 isset() 或 empty() 时调用
public void __unset ( string $name )  // 当对不可访问属性调用 unset() 时调用
```
**属性重载只能在对象中进行,所以静态属性没有重载的魔术方法**

举个例子:

```php
<?php
class A
{
    private $data = array();
    private $private;
 
    public function __set($name , $value)
    {
        echo "set {$name} to {$value}\n";
        // 存下设置的键值
        $this->data[$name] = $value;
    }
 
    public function __get($name)
    {
        echo "get {$name}\n";
        if (array_key_exists($name, $this->data)) {
            return $this->data[$name];
        }
        return null;
    }
 
    public function __isset($name)
    {
        echo "isset {$name}?\n";
        return isset($this->data[$name]);
    }
 
    public function __unset($name)
    {
        echo "unset {$name}\n";
        unset($this->data[$name]);
    }
}
 
$a = new A();
 
// 访问无定义的属性会触发魔术方法
$a->a = 'a';  // output: set a to a
$a->a;  // output: get a
isset($a->a);  // output: isset a?
unset($a->a);  // output: unset a
 
// 访问private也会触发魔术方法
$a->private = 'private';  // output: set private to private
$a->private;  // output: get private
isset($a->private);  // output: isset private?
unset($a->private);  // output: unset private
```

### 方法重载

方法重载的魔术方法有:

```php
public mixed __call ( string $name , array $arguments )  // 在对象中调用一个不可访问方法时调用
public static mixed __callStatic ( string $name , array $arguments )  // 用静态方式中调用一个不可访问方法时调用
```

举个例子:
 
```php
 <?php
class A
{
    private function getSecret() {}
    private static function getStaticSecret() {}
 
 
    public function __call($name, $args)
    {
        echo "call {$name} function with arguments(" . implode(',', $args) . ")\n";
    }
 
    public static function __callStatic($name, $args)
    {
        echo "call static {$name} function with arguments(" . implode(',', $args) . ")\n";
    }
}
 
$a = new A();
 
// 用静态方法访问private方法
A::getSecret(1, 2, 3);  // output: call static getSecret function with arguments(1,2,3)
 
// 用非静态方法访问private方法
$a->getSecret(1, 2, 3);  // output: call getSecret function with arguments(1,2,3)
 
// 用静态方法访问不存在的方法
A::getNone(4, 5, 6);  // output: call static getNone function with arguments(4,5,6)
 
// 用非静态方法访问不存在的方法
$a->getNone(4, 5, 6); // output: call getNone function with arguments(4,5,6)
```

## 对象与引用

在PHP5+中,每个对象都有代表这个对象的标识符,而对象变量存储的是代表这个对象的标识符,在使用的时候我们会对这个标识符进行复制和引用,我们要好好对 **复制** 和 **引用** 进行区分.

* 当对象作为参数传递,作为结果返回,和赋值给另一个变量的时候,标识符会进行复制.复制指的是有多一个有相同标识符值的变量.
* 当先是使用 ```&``` 进行变量操作的时候,变量之间的关系是引用.引用就是一个别名,它们使用同一个标识符.

举个例子:

```php
<?php
class A {}
 
$a = new A();  // ($a) -> 标识符1 -> 对象1
 
// 复制标识符
$b = $a;  // ($a), ($b) -> 标识符1 -> 对象1
 
// 引用标识符
$c = &$a;  // ($a, $c), ($b) -> 标识符1 -> 对象1
 
unset($a);  // ($c), ($b) -> 标识符1 -> 对象1
unset($c);  // (), ($b) -> 标识符1 -> 对象1
// 此时没有任何变量引用对象1,对象1会被回收
$b = null;  // () -> 标识符1 -> 对象1
```

再举个复杂点的例子:

```php
<?php
class Foo {
  public static $used = 0;
  public $id;
  public function __construct() {
    $this->id = self::$used++;
  }
  public function __clone() {
    $this->id = self::$used++;
  }
}
 
$a = new Foo();
// $b是$a指向的标识符的拷贝
$b = $a;
// $c是$a的一个别名
$c = &$a;
 
// 都是指向同一个对象
var_dump($a->id);  // output: 0
var_dump($b->id);  // output: 0
var_dump($c->id);  // output: 0
 
var_dump(Foo::$used);  // output: 1
 
$a = new Foo();
 
// 指向新的对象
var_dump($a->id);  // output: 1
// 使用的是旧的标识符所以指向旧的对象
var_dump($b->id);  // output: 0
// 作为别名和$a一样指向新的对象
var_dump($c->id);  // output: 1
 
var_dump(Foo::$used);  // output: 2
 
unset($a);
 
var_dump($b->id);  // output: 0
// 作为别名的$a,所以$c直接使用新的标识符,指向新的对象
var_dump($c->id);  // output: 1
 
 
// $a作为$b的别名置null
$a = &$b;
$a = null;
 
// 大家都为null
var_dump($a);  // output: null
var_dump($b);  // output: null
 
// 又新建一个新的对象
$a = clone $c;
 
var_dump($a->id);  // output: 2
 
var_dump(Foo::$used);  // output: 3
 
// 这个时候id为1的对象已经没有变量引用它,这个对象会被垃圾回收器回收
unset($c);
```