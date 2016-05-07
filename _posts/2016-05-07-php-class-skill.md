---
title: "【PHP拾遗】系列-类(class)-技巧"
category: [tech]
tag: [php]
---


PHP的类有一些技巧,掌握他们可以让设计更加优雅而且提高工作效率.

## 对象序列化

> 序列化： 将数据结构或对象转换成二进制串的过程
> 反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程

在php中所有的值都可以使用函数 ```serialize()``` 来转换成一个包含字节流的字符串,这个过程就叫序列化.

通过函数 ```unserialize()``` 序列化后的字符串转换回php原来的值,这个过程叫反序列化.

**在PHP中要反序列化一个对象,这个对象的类必须在之前已经定义过.**
我们举个例子:

```php
<?php
class A
{
    private $private = 'private';
    public $public = 'public';
    protected $protected = 'protected';
 
    public function showIntro() {
        echo 'i am Nutto';
    }
}
 
$a = new A();
 
// 序列化$a,获得字符串
$serialized_a = serialize($a);
print_r($serialized_a);  // output: O:1:"A":3:{s:10:"Aprivate";s:7:"private";s:6:"public";s:6:"public";s:12:"*protected";s:9:"protected";}
 
// 反序列化,获取对象
$b = unserialize($serialized_a);
print_r($b);  // output: A Object ( [private:A:private] => private [public] => public [protected:protected] => protected )
```

对于序列化我们在PHP中比较常接触的例子就是SESSION的使用,当我们使用 ```session_start() ``` 函数的时候,SESSION会在从会话数据存放的地方将数据抽出然后经过反序列化后存放在 ```$_SESSION```超级全局变量中,而在每次会话结束的时候会将会话的数据序列化放到数据存放的地方.

## 魔术方法

PHP 将所有以 ``` __``` (两个下划线)开头的类方法保留为魔术方法,所以在定义类方法时,除了上述魔术方法,建议不要以 ``` __ ``` 为前缀.

魔术方法有例如:

|        方法名        |                     调用时机                    |
|:--------------------:|:-----------------------------------------------:|
|  ```__construct()``` |                  对象构造时调用                 |
|  ```__destruct()```  |                  对象析构时调用                 |
|    ```__call()```    |           调用对象不可访问的方法时调用          |
| ```__callStatic()``` |     用静态的方式调用对象不可访问的方法时调用    |
|     ```__get()```    |           访问对象不可访问的属性时调用          |
|     ```__set()```    |           设置对象不可访问的属性时调用          |
|    ```__isset()```   | 使用 ```isset()``` 检测不可访问的属性的时候调用 |
|    ```__unset()```   | 使用 ```unset()``` 移除不可访问的属性的时候调用 |
|    ```__sleep()```   |   使用 ```serialize()``` 序列化对象的时候调用   |
|   ```__wakeup()```   | 使用 ```unserialize()``` 反序列化对象的时候调用 |
|  ```__toString()```  |          当对象当作字符串输出的时候调用         |
|   ```__invoke()```   |           当对象当作函数调用的时候调用          |
|  ```__set_state()``` |      使用 ```var_export()``` 输出对象时调用     |
|    ```__clone()```   |       使用 ```clone``` 复制对象的时候调用       |
|  ```__debugInfo()``` |       使用 ```var_dump()``` 输出对象时调用      |

其中有一些的魔术方法已经在[【PHP拾遗】系列-类(class)-进阶](http://blog.nuttopan.cn/blog/php-class-advance/)有所涉及,现在我们就说一下一些没有涉及到的魔术方法.

### ```__sleep()``` 和 ```__wakeup()```

在序列化的时候, ```__sleep()``` 这个魔术方法会被调用,这个方法要返回一个包含对象中所有应被序列化的变量名称的数组,如果没有返回任何内容则会发生错误.

在反序列化的时候 ```__wakeup()``` 会被调用
举个例子:

```php
<?php
class A
{
    private $private = 'private';
    public $public = 'public';
    protected $protected = 'protected';
 
    public function showIntro() {
        echo 'i am Nutto';
    }
 
    public function __sleep() {
        echo "call __sleep\n";
        return array('private');
    }
 
    public function __wakeup() {
        echo "call __wakeup\n";
    }
}
 
$a = new A();
 
// 序列化会唤起__sleep()
$serialized_a = serialize($a);  // output: call __sleep
 
// 对象只序列化了__sleep()返回的属性列表对应的属性
print_r($serialized_a);  // output: O:1:"A":1:{s:10:"Aprivate";s:7:"private";}
 
// 反序列化会唤起__wakeup()
print_r(unserialize($serialized_a));  // output: call __wakeup
```

### ```__toString()``` 

当对象被当作字符串输出的时候就会唤起对象中的 ```__toString()``` 方法
比如:

```php
<?php
class A
{
    public function __toString() {
        return __METHOD__;
    }
}
 
$a = new A();
 
// 当作字符串输出
echo $a;  // output: A::__toString
 
// 使用%s也是当作字符串输出
printf("%s", $a);  // output: A::__toString
 
// 使用%f当作浮点输出的时候就会出问题
printf("%f", $a);  // output: Notice: Object of class A could not be converted to float
```

### ```__invoke()```

当对象被当作方法调用的时候, ```__invoke()``` 就会唤起
比如:

```php
<?php
class A
{
    public function __invoke() {
        echo 'invoke ' . __METHOD__;
    }
}
 
$a = new A();
 
// 当对象被当作函数调用的时候__incoke会被唤起
$a();  // output: invoke A::__invoke
```

### ```__set_state()```

```static object __set_state ( array $properties )```

这个方法是一个静态方法,会传入一个参数是 ```var_export()``` 传入的第一个参数,对于对象来说就是 array('property' => value, ...) 格式排列的类属性

当调用 ```var_export()``` 输出对象的时候,该方法会被唤起

```php
<?php
class A
{
    private $private = 'private';
    public static function __set_state($properties) {
        echo 'invoke ' . __METHOD__ . "\n";
        return $properties;
    }
}
 
$a = new A();
 
// 当对象被当作函数调用的时候__incoke会被唤起
var_export($a);  // output: A::__set_state(array(
                 //    'private' => 'private',
                 // ))
```

### ```__debugInfo()```

当对象被 ```var_dump()``` 调用的时候, ```__dubugInfo()``` 会被唤起.
比如:

```php
<?php
class A
{
    public function __debugInfo() {
        echo 'degug ' . __METHOD__ . "\n";
        return array();
    }
}
 
$a = new A();
 
// 当对象被当作函数调用的时候__incoke会被唤起
var_dump($a);  // output: degug A::__debugInfo
               //   object(A)#1 (0) {
               // }
```

### ```__clone()```

当使用 ```clone()``` 进行对象复制的时候, ```__clone()``` 方法会被唤起.

当使用 ```clone()``` 进行对象复制的时候,执行的是一个浅复制,即是说如果对象中有引用属性的话,复制出来的对象的该属性也会引用到原来的对象.

辛好我们可以通过 ```clone()``` 唤起的 ```__clone()```魔术方法,对引用属性进行进一步的复制.

比如:

```php
<?php
class SubObject
{
    static $instances = 0;
    public $instance;
 
    public function __construct() {
        $this->instance = ++self::$instances;
    }
 
    public function __clone() {
        $this->instance = ++self::$instances;
    }
}
 
class MyCloneable
{
    public $object1;
    public $object2;
 
    function __clone()
    {
 
        // 强制复制一份this->object， 否则仍然指向同一个对象
        $this->object1 = clone $this->object1;
    }
}
 
$obj = new MyCloneable();
 
$obj->object1 = new SubObject();
$obj->object2 = new SubObject();
 
$obj2 = clone $obj;
 
 
print("Original Object:\n");
print_r($obj);
/*
output:
Original Object:
MyCloneable Object
(
    [object1] => SubObject Object
        (
            [instance] => 1
        )
 
    [object2] => SubObject Object
        (
            [instance] => 2
        )
 
)
*/
 
print("Cloned Object:\n");
print_r($obj2);
/*
output:
Cloned Object:
MyCloneable Object
(
    [object1] => SubObject Object
        (
            [instance] => 3
        )
 
    [object2] => SubObject Object
        (
            [instance] => 2
        )
 
)
*/
```

## 对象的比较

* 当使用比较运算符 ```==``` 比较两个对象变量时,比较的原则是:如果两个对象的属性和属性值 都相等,而且两个对象是同一个类的实例,那么这两个对象变量相等.
* 当使用全等运算符 ```===``` ,这两个对象变量一定要指向某个类的同一个实例(即同一个对象).

```php
<?php
class A
{
    public $var;
 
    public function __construct($var) {
        $this->var = $var;
    }
}
 
class B
{
    public $var;
 
    public function __construct($var) {
        $this->var = $var;
    }
}
 
$a = new A('a');
$shadow_a = $a;
 
$mock_a = new A('a');
$fake_a = new A('b');
 
$b = new B('b');
$fake_b = new B('a');
 
// 同类同值
var_dump($a == $mock_a);  // output: true
// 同类不同值
var_dump($a == $fake_a);  // output: false
// 不同类同值
var_dump($a == $fake_b);  // output: false
// 不同类不同值
var_dump($a == $b);  // output: false
 
// 同对象
var_dump($a === $shadow_a);  // output: true
// 同类同值
var_dump($a === $mock_a);  // output: false
// 同类不同值
var_dump($a === $fake_a);  // output: false
// 不同类同值
var_dump($a === $fake_b);  // output: false
// 不同类不同值
var_dump($a === $b);  // output: false
```

## 类型约束

在PHP5+我们可以指定参数的类型来进行类型约束

类型可以指定为:

* object name
* interface
* array
* callable

如果默认值为 ```null``` 那么传入的实参也可以为 ```null```

如果一个类或接口指定了类型约束,则其所有的子类或实现也都如此.

举个例子:

```php
<?php
class A {};
 
function testClass(A $a_obj) {};
 
function testArray(array $array) {};
 
function testInterface(Traversable $iterator) {};
 
function testCallable(callable $callabel) {};
 
function test() {};
 
// 正常调用
testClass(new A());  // 传入一个A的对象
testArray(array());  // 传入数组
testInterface(new ArrayObject(array()));  // 传入可以遍历的对象
testCallable('test');  // 传入可调用函数的名字
 
 
// 非正常调用
testClass('asdf');  // output:  Fatal error:  Uncaught TypeError: Argument 1 passed to testClass() must be an instance of A
testArray('asdf');  // output: Fatal error:  Uncaught TypeError: Argument 1 passed to testArray() must be of the type array
testInterface('asdf');  // output: Fatal error: Uncaught TypeError: Argument 1 passed to testInterface() must implement interface Traversable
testCallable('asdf');  // output: Fatal error: Uncaught TypeError: Argument 1 passed to testCallable() must be callable
```

## 对象遍历

在PHP5+中,默认的时候遍历对象会遍历所有可见的属性

```php
<?php
class A {
    private $private = 'private';
    public $public = 'public';
    protected $protected = 'protected';
};
 
$a = new A();
 
// output: public => public
foreach ($a as $key => $value) {
    echo "{$key} => {$value}";
}
```

我们也可以通过实现**Iterator**接口来决定遍历的方法,详情参考[SPL的迭代器](http://php.net/manual/zh/spl.iterators.php)