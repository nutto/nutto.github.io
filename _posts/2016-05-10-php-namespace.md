---
title: "【PHP拾遗】系列-命名空间"
category: [tech]
tag: [php]
---


在编写大规模程序和涉及多个文件的程序的时候很容易遇到以下的两个问题:

* 用户编写的代码与PHP内部的类/函数/常量或第三方类/函数/常量之间的名字冲突。
* 为很长的标识符名称(通常是为了缓解第一类问题而定义的)创建一个别名（或简短）的名称，提高源代码的可读性。

在PHP中我们可以使用命名空间来解决这些问题

## 定义

* 只有以下类型受命名空间的影响
    * 类(Classes或Traits)
    * 接口(Interfaces)
    * 函数(Functions)
    * 常量(Constants)
* 除 ```declare``` 关键词外,命名空间一定要在所有代码之前
* 同一个命名空间可以定义在多个文件中
* 可以定义子的命名空间
* 可以在同一个文件中定义多个命名空间

举个例子:

```php
<?php
namespace A {
    trait TraitTest { /* ... */ }
    interface testInterface { /* ... */ }
    const CONNECT_OK = 1;
    class Connection { /* ... */ }
    function connect() { /* ... */  }
}
 
// 同一个文件可以有多个命名空间(不推荐)
namespace A\subnamespace {  // 带有子的命名空间
    /* ... */
}
```

## 使用

### 分类

|     类别     |       描述      |                   调用例子                  |                    实际调用                   |
|:------------:|:---------------:|:-------------------------------------------:|:---------------------------------------------:|
|  非限定名称  |      无前缀     |               ```new foo()```               |        ```new CURRENTNAMESPACE\foo()```       |
|   限定名称   | 有前缀,相对路径 |         ```new subnamespace\foo()```        | ```new CURRENTNAMESPACE\subnamespace\foo()``` |
| 完全限定名称 | 全路径,绝对路劲 | ```new \mainnamespace\subnamespace\foo()``` |  ```new \mainnamespace\subnamespace\foo()```  |

### 使用变量动态访问元素

使用变量动态访问元素的时候**必须使用完全限定**,所以一定要加上访问成员的命名空间前缀,否则就只会访问到全局的命名空间成员.

举个例子:

**global.php**

```php
<?php
class ClassName
{
    public function __construct() {
        echo 'global-', __METHOD__, "\n";
    }
}
 
function functionName() {
    echo 'global-', __METHOD__, "\n";
}
 
const CONSTANT = 'global';
```

**index.php**

```php
<?php
namespace A;
 
include_once 'global.php';
 
class ClassName
{
    public function __construct() {
        echo 'A-', __METHOD__, "\n";
    }
}
 
function functionName() {
    echo 'A-', __METHOD__, "\n";
}
 
const CONSTANT = 'index';
 
// 无限定
$unqualified_class = 'ClassName';
$unqualified_function = 'functionName';
$unqualified_const = 'CONSTANT';
 
// 由于没有限定,使用的都是全局命名空间下的成员
new $unqualified_class();  // output: global-ClassName::__construct
$unqualified_function();  // output: global-functionName
echo constant($unqualified_const);  // output: global
 
 
// 有限定
$qualified_class = 'A\ClassName';
$qualified_function = 'A\functionName';
$qualified_const = 'A\CONSTANT';
 
// 使用限定的话就会访问对应命名空间的成员(必须使用完全
// 限定,所以无必要在命名空间的最前面增加\)
new $qualified_class();  // output: A-A\ClassName::__construct
$qualified_function();  // output: A-A\functionName
echo constant($qualified_const);  // output: index
```

### 获取当前命名空间

有两个方法可以获取当前的命名空间名称

* 魔术常量 ```__NAMESPACE```
* 关键字 ```namespace```


 **```__NAMESPACE__``` 可以作为字符输出**

```php
<?php
namespace A;
 
echo __NAMESPACE__;  // output: A
```

**```namespace```可以作为命名空间的构成部分,用来代指现在的命名空间**

```php
<?php
namespace A;
 
const CONSTANT = 'A';
 
// 输出的成员是A\CONSTANT
echo namespace\CONSTANT;
```

### 使用 ```use``` 来导入命名空间或设置别名

我们可以使用 ```use``` 来导入某个命名空间(**注意:导入命名空间的意思是这个命名空间可以使用了,并没有导入对应的文件,要导入对应的文件还是要使用 ```include``` 等的方法, 或者直接使用自动加载的机制**),或者为这个命名空间设置一个别名.

```php
<?php
namespace MainNameSpace\SubNameSpace\B;
 
const CONSTANT = 'B';
 
namespace A;
use MainNameSpace\SubNameSpace\B as B;  // 导入命名空间并设置别名
 
const CONSTANT = 'A';
 
echo CONSTANT;  // output: A
// 只需要使用别名就可以访问对应的命名空间成员
echo B\CONSTANT;  // output: B
```