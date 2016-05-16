---
title: "【PHP拾遗】系列-8个有用的预定义魔术常量"
category: [tech]
tag: [php]
---


在PHP中有很多预定以的常量,其中有8个魔术常量非常有用.他们基本上都是用于反馈自身位置的信息

|         名称        |          描述          |
|:-------------------:|:----------------------:|
|    ```__LINE__```   |        当前行号        |
|    ```__FILE__```   | 文件的完整路径和文件名 |
|    ```__DIR__```    |     文件所在的目录     |
|  ```__FUNCTION__``` |      当前函数名称      |
|   ```__CLASS__```   |      当前类的名称      |
|   ```__TRAIT__```   |    当前Trait 的名字    |
|   ```__METHOD__```  |     当前类的方法名     |
| ```__NAMESPACE__``` |   当前命名空间的名称   |

```php
<?php
namespace TestNamespace;
 
trait TestTrait
{
    public static function traitMethod()
    {
        var_dump(__TRAIT__);
    }
}
 
class ClassA
{
    use TestTrait;
    public static function classMethod()
    {
        var_dump(__METHOD__);
        var_dump(__CLASS__);
        var_dump(__FUNCTION__);
    }
}
 
function funcA()
{
    var_dump(__FUNCTION__);
}
funcA();  // output: TestNamespace\funcA
 
ClassA::classMethod();
/*output:
TestNamespace\ClassA::classMethod
TestNamespace\ClassA
classMethod
*/
ClassA::traitMethod();  // output: TestNamespace\TestTrait
 
var_dump(__LINE__);  // output: 37
var_dump(__FILE__);  // output: G:\projects\php\index.php
var_dump(__DIR__);  // output: G:\projects\php
var_dump(dirname(__FILE__));  // output: G:\projects\php
var_dump(__NAMESPACE__);  // output: TestNamespace
```