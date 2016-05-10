---
title: "【PHP拾遗】系列-函数"
category: [tech]
tag: [php]
---


函数作为一个语言基本的语法结构,,很有必要做一些了解

本文从四个方面讲述一下PHP的函数:

* 定义
* 使用和传参
* 返回
* 需要注意的方面

## 定义

* 除非定义是有条件的,否则函数无需在调用前定义
* 定义过的函数和类都有全局的作用域,可以在任何地方调用
* 函数或类中都可以定义函数
* 函数不能重载,也不能取消或重定义

举个例子:

```php
<?php
 
// 可以在定义前调用
test();  // output: i am test
function test() {
    echo "i am test\n";
}
 
 
// 只要定义过的函数都可以使用
function outer() {
    // 函数中可以定义函数
    function inner() {
        echo "i am inner\n";
    }
}
outer();  // 定义一下inner
inner();  // output: i am inner 
```

## 使用和传参

有两种传参的方式分别是**值传参**和**引用传参**



* **值传参**

***函数默认的传参方式就是值传参***

```php
<?php
 
$origin = 'origin';
 
function test($string) {
    // 改变传入的值
    $string = 'changed';
}
 
test($origin);

// 原来的值没有改变
echo $origin;  // output: origin
```

* **引用传参**


```php
<?php
 
$origin = 'origin';
 
// 改为引用传参
function test(&$string) {
    // 改变传入的值
    $string = 'changed';
}
 
test($origin);
 
// 原来的值发生改变
echo $origin;  // output: changed
```

### 默认值

值参数可以使用默认值,引用参数在php5+也可以使用默认值

```php
<?php
function test(&$string='asdf', $string2='1234') {};
 
test();
```

### 类型约束

在php5+后参数也可以声明类型约束

```php
<?php
function test(float $float) {};
 
test(1.0);
 
// 会发生错误
test('asdf');
```

可以使用的类型约束有

|         类型         |      描述      |
|:--------------------:|:--------------:|
| Class/Interface name |   类或接口名   |
|         array        |      数组      |
|       callable       | 可调用的函数名 |
|         bool         |     布尔值     |
|         float        |     浮点数     |
|          int         |      整型      |
|        string        |     字符串     |

## 返回

* 一次只可以返回一个值,但可以通过返回数组达到返回多个值的效果
* 无定义的返回会返回 ```null```
举个例子:


```php
<?php
function test() {};
 
$null = test();
 
// 无定义返回就返回null
echo $null == null;  // output: true
 
 
function hasReturn() {
    return array('a', 'b');
}
 
var_dump(hasReturn());
/*output:
array(2) {
  [0]=>
  string(1) "a"
  [1]=>
  string(1) "b"
}
*/
```

## 要注意的要点

### 变量函数

如果一个变量名后面紧跟一对圆括号 ```()``` 那么PHP会尝试寻找与该变量的值相同的函数,并尝试执行它.

```php
<?php
function test() {
    echo 'i am test';
};
 
function actFunc($name) {
    // 使用变量函数前可以先做一下可用性校验
    if (function_exists($name) && is_callable($name)) {
        // 尝试执行与该变量值同名的函数
        $name();
    }
}
actFunc('test');  // output: i am test
```

### 可变数量参数

使用下面的函数,可以在函数里面实现可变长度的函数传值

|        函数名       |        描述        |
|:-------------------:|:------------------:|
| ```func_num_args``` |   获取参数的数量   |
|  ```func_get_arg``` |   获取指定的参数   |
| ```func_get_args``` | 获取参数列表的数组 |

举个例子:

```php
<?php
function test() {
    // 获取参数数量
    $args_num = func_num_args();
    // 获取所有的参数
    $args = func_get_args();
    if ($args_num > 0) {
        // 如果可以的话,获取第一个参数
        $arg0 = func_get_arg(0);
        var_dump($arg0);
    }
    var_dump($args);
    var_dump($args_num);
};
test('t', 'e', 's', 't');
/*output:
string(1) "t"
array(4) {
  [0]=>
  string(1) "t"
  [1]=>
  string(1) "e"
  [2]=>
  string(1) "s"
  [3]=>
  string(1) "t"
}
int(4)
*/
```

### 闭包函数

* 闭包函数可以使用 ```use``` 关键词从父作用域中继承变量

```php
<?php
 
$msg = 'hello';
 
$example = function() use($msg) {
    var_dump($msg);
};
 
$example();  // output: hello
 
// 上面的use结构定义的是值传参所以不会跟随父作用域的变量变化而变化
$msg = 'world';
$example();  // output: hello
```