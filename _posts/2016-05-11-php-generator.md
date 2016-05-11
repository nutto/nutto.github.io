---
title: "【PHP拾遗】系列-生成器"
category: [tech]
tag: [php]
---


PHP的迭代器对于小型的迭代来说还是显得太过臃肿,辛好PHP的生成器提供了一种更容易的方法来产生可以迭代的对象.

## 使用 ```yield``` 来产生需要迭代的数据

PHP的通过将函数和 ```yield``` 关键字结合来建造一个生成器

```php
<?php
function genOneToThree() {
    for ($i = 1; $i <= 3; $i++) {
        yield $i;
    }
}
 
foreach (genOneToThree() as $value) {
    echo "$value\n";
}
/* output:
1
2
3
*/
```

要注意的是,生成器内不能使用 ```return``` 返回数值,但是可以使用空的 ```return``` 来代表生成器的结束

```yield``` 看起来很像 ```return``` 不过不同之处在于普通 ```return``` 会返回值并终止函数的执行, 而 ```yield``` 会返回一个值给循环调用此生成器的代码并且只是暂停执行生成器函数.

### ```yield``` 也可以返回一个键值对

```php
<?php
$input = <<<'EOF'
1;PHP;Likes dollar signs
2;Python;Likes whitespace
3;Ruby;Likes blocks
EOF;
 
function input_parser($input) {
    foreach (explode("\n", $input) as $line) {
        $fields = explode(';', $line);
        $id = array_shift($fields);
 
        yield $id => $fields;
    }
}
 
foreach (input_parser($input) as $id => $fields) {
    echo "$id:\n";
    echo "    $fields[0]\n";
    echo "    $fields[1]\n";
}
/*output:
1:
    PHP
    Likes dollar signs
2:
    Python
    Likes whitespace
3:
    Ruby
    Likes blocks
*/
```

### ```yield``` 可以使用引用来生成值

```php
<?php
function &gen_reference() {
    $value = 3;
 
    while ($value > 0) {
        yield $value;
    }
}
 
// $number是$value的一个引用,所以$number的改变会影响到$value
foreach (gen_reference() as &$number) {
    echo (--$number).'... ';
}
/*output:
2... 1... 0...
*/
```

### 可以使用 ```yield from``` 来遍历数组,迭代器和调用函数

```php
<?php
function countToTen()
{
    yield 1;
    yield 2;
    yield from [3, 4];
    yield from yieldFiveToSeven();
    yield from new ArrayIterator([8, 9]);
    yield 10;
}
 
function yieldFiveToSeven()
{
    yield 5;
    yield from new ArrayIterator([6, 7]);
}
 
foreach (countToTen() as $number) {
    echo $number, ' ';
}
/*output:
1 2 3 4 5 6 7 8 9 10
*/
```