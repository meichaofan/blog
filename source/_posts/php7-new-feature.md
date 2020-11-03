---
title: php7新特性
date: 2019-07-10 08:36:05
tags: PHP
---

### PHP7新特性

#### 太空船操作符 <=>

- 太空船操作符用于比较两个表达式

例如，当$a小于、等于或大于$b时，它分别返回-1、0或1

```
echo 1<=>1; //0
echo 1<=>2; //-1
echo 2<=>1; //1
```

#### 类型声明

- declare(strict_types=1); // strict_types=1表示开启严格模式

```
declare(strict_types=1); //必须在脚本开始出声明
function add(int ...$arg): int
{
    return array_sum($arg);
}
```

#### null合并操作符

```
$page = isset($_GET('page')) ? $_GET['page']:0；
改成
$page = $_GET('page') ?? 0;
```
**注意**，这里是`isset($val)`

#### 常量数组

- 不可以修改

```
define('ANIMAL', ['dog', 'cat', 'bird']);
```

#### NameSpace批量导入

```
use Space\{ClassA,ClassB,ClassC as C};
```

#### try...catch 捕获 error 

```
try {
    undefinedfunc();
} catch (Error $e) {
    var_dump($e);
}

也可以
set_exception_handler(
    function ($e) {
        var_dump($e);
    }
);
undefinedfunc();
```

PHP7返回结果

```
class Error#1 (8) {
  protected $message =>
  string(42) "Call to undefined function undefinedfunc()"
  private $string =>
  string(0) ""
  protected $code =>
  int(0)
  protected $file =>
  string(41) "/home/meichaofan/PHP7/capter02/inedx2.php"
  protected $line =>
  int(4)
  private $trace =>
  array(0) {
  }
  private $previous =>
  NULL
  public $xdebug_message =>
  string(199) "
Error: Call to undefined function undefinedfunc() in /home/meichaofan/PHP7/capter02/inedx2.php on line 4

Call Stack:
    0.0001     387432   1. {main}() /home/meichaofan/PHP7/capter02/inedx2.php:0
"
}
```

#### Closure::call

```
class Test
{
    private $num = 1;
}

$f = function () {
    return $this->num + 1;
};

echo $f->call(new Test());
```

#### intdiv 函数

```
echo intdiv(10,3); //整除 == 3
```

#### list 的方括号写法

```
之前
$arr = [1,2,3];
list($a,$b,$c) = $arr;
现在可以这么写
$arr = [1,2,3];
[$a,$b,$c] = $arr;
```

#### 抽象语法树(AST)

```
($a)['b'] = 1;
PHP5 会报错
PHP7 因为AST，不会报错
```
