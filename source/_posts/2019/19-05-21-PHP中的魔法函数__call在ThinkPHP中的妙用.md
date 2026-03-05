---
title: PHP中的魔法函数 “__call” 在ThinkPHP中的妙用
date: 2019-05-21 17:27:20
comments: true
tags:
- PHP
- ThinkPHP
- __call
- 魔法函数
- 控制器
- 方法调用
- _post
- _get
- EmptyAction
- 框架设计
categories:
- 软件开发
- PHP编程
---

PHP 中的 `__call` 方法在 ThinkPHP 中被广泛使用，用于处理未定义的方法调用。在 ThinkPHP 中，`__call` 方法通常用于实现类似于 `_post` 和 `_get` 方法的功能，即获取 `$_POST` 或 `$_GET` 数组中的数据。

## 1. `__call` 方法的定义

在 ThinkPHP 中，`__call` 方法通常定义在控制器类中，用于处理未定义的方法调用。例如：

```php
class IndexController extends Controller {
    public function __call($method, $args) {
        // 处理未定义的方法调用
    }
}

// 调用未定义的方法
$controller = new IndexController();
$controller->undefinedMethod(); // 会触发 __call 方法
```

我们以GET和POST为例，写一个相对完整的例子：

以下是一个使用 `__call` 实现类似功能的示例：
```php
class MyController {
    public function __call($method, $args) {
        if (strpos($method, '_') === 0) {
            $type = substr($method, 1);
            if (in_array($type, ['post', 'get'])) {
                $name = isset($args[0]) ? $args[0] : '';
                $filter = isset($args[1]) ? $args[1] : null;
                $default = isset($args[2]) ? $args[2] : '';
                $input = ($type === 'post') ? $_POST : $_GET;
                $value = isset($input[$name]) ? $input[$name] : $default;
                if ($filter) {
                    $value = call_user_func($filter, $value);
                }
                return $value;
            }
        }
        trigger_error("Call to undefined method " . __CLASS__ . "::" . $method . "()", E_USER_ERROR);
    }
}

$controller = new MyController();
$postValue = $controller->_post('username');
$getValue = $controller->_get('id');
```
在上述示例中，`__call` 方法用于处理未定义的方法调用。当调用未定义的方法时，会根据方法名的前缀判断是获取 `$_POST` 还是 `$_GET` 数据，然后进行相应的处理。如果方法名不符合预期，会触发一个错误。

需要注意的是，`__call` 方法的实现可能会因具体的需求而有所不同。在实际应用中，你可能需要根据项目的具体情况来调整 `__call` 方法的实现。


## 2. `_post` 和 `_get` 方法的实现

在 ThinkPHP 中，`_post` 和 `_get` 方法通常用于获取 `$_POST` 和 `$_GET` 数组中的数据。

如果不考虑魔法函数__call，对于这两个函数的实现方式如下：

```php
class IndexController extends Controller {
    public function _post($name, $default = '') {
        return isset($_POST[$name]) ? $_POST[$name] : $default; 
    } 
    public function _get($name, $default = '') {
        return isset($_GET[$name])? $_GET[$name] : $default; 
    }

}

// 获取 POST 参数
$controller = new IndexController();
$username = $controller->_post('username'); // 获取 $_POST['username'] 的值

// 获取 GET 参数
$id = $controller->_get('id'); // 获取 $_GET['id'] 的值
```

而我们实现这两个函数，还会考虑对GET和POST参数值的过滤，以及默认值的设置。标准的 ThinkPHP 3.1.2 较新的版本里，`_post` 和 `_get` 方法定义于 `ThinkPHP/Library/Think/Controller.class.php` 文件中，代码如下：

```php
/**
 * 获取POST参数 支持过滤和默认值
 * @param string $name 变量名
 * @param string $filter 过滤方法
 * @param mixed $default 默认值
 * @return mixed
 */
public function _post($name='',$filter=null,$default='') {
    return $this->input('post',$name,$filter,$default);
}

/**
 * 获取GET参数 支持过滤和默认值
 * @param string $name 变量名
 * @param string $filter 过滤方法
 * @param mixed $default 默认值
 * @return mixed
 */
public function _get($name='',$filter=null,$default='') {
    return $this->input('get',$name,$filter,$default);
}
```
这些方法借助调用 `input` 方法从 `$_POST` 或 `$_GET` 数组中获取数据，同时支持数据过滤和默认值设置。



## 3. `__call` 实现原理

基于以上代码，我们不难发现，__call有以下这些规则：

- **触发条件**：当调用一个对象中不存在的方法时，PHP 会自动调用该对象的 `__call` 方法。
- **参数传递**：`__call` 方法接收两个参数，`$method` 表示调用的方法名，`$args` 表示传递给该方法的参数数组。
- **方法处理**：在 `__call` 方法中，可以根据方法名和参数来实现相应的逻辑。例如，在上述示例中，根据方法名的前缀判断是获取 `$_POST` 还是 `$_GET` 数据，然后进行相应的处理。



## 4. 用魔法战胜魔法

跟  __call 这个魔法函数类似，ThinkPHP 3.1.2中我们实现EmptyAction.class.php，即找不到对应的Action或者Controller的时候，代码逻辑走到这里。于是，`EmptyAction.class.php` 这个特殊的类就出现了。当系统找不到对应的 `Action` 或者 `Controller` 时，代码逻辑就会走到这里。

### 实现原理
ThinkPHP 框架在接收到请求后，会依据请求的 URL 来查找对应的 `Controller` 和 `Action`。若找不到匹配的 `Controller` 或者 `Action`，框架就会尝试调用 `EmptyAction.class.php` 中的方法。

### 使用方法
要使用 `EmptyAction.class.php`，你需要在项目的 `Lib/Action` 目录下创建这个文件，并且定义一个继承自 `Action` 的 `EmptyAction` 类。

### 代码示例
以下是 `EmptyAction.class.php` 的一个简单示例：
```php
<?php
// 引入 ThinkPHP 的 Action 类
import('@.Action.BaseAction');

class EmptyAction extends Action {
    // 当找不到对应的 Action 时，会调用这个方法
    public function _empty() {
        // 输出提示信息
        $this->error('你访问的页面不存在！');
    }

    // 当找不到对应的 Controller 时，会调用这个方法
    public function index() {
        // 输出提示信息
        $this->error('你访问的控制器不存在！');
    }
}
?>
```

