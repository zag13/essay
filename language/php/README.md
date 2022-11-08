# PHP

## 面向对象

### 核心概念

- **类** − 定义了一件事物的抽象特点。类的定义包含了数据的形式以及对数据的操作。
- **对象** − 是类的实例。
    - 对象就是数据，对象本身不包括方法。但是对象有一个“指针”指向一个类，这个类中有方法。
    - 方法描述不同属性所产生的不同表现。
    - 类和对象是不可分割的。除非使用`(object)`强制转换，这是PHP中的`stdClass`会收留这个对象。
- **成员变量** − 定义在类内部的变量。该变量的值对外是不可见的，但是可以通过成员函数访问，在类被实例化为对象后，该变量即可称为对象的属性。
- **成员函数** − 定义在类的内部，可用于访问对象的数据。
- **继承** − 继承性是子类自动共享父类数据结构和方法的机制，这是类之间的一种关系。在定义和实现一个类的时候，可以在一个已经存在的类的基础之上来进行，把这个已经存在的类所定义的内容作为自己的内容，并加入若干新的内容。
- **父类** − 一个类被其他类继承，可将该类称为父类，或基类，或超类。
- **子类** − 一个类继承其他类称为子类，也可称为派生类。
- **多态** − 多态性是指相同的函数或方法可作用于多种类型的对象上并获得不同的结果。不同的对象，收到同一消息可以产生不同的结果，这种现象称为多态性。
    - 多态指同一类对象在运行时的具体化。
    - PHP中的子类无法向上转型为父类，从而失去多态最典型的特征。
    - 多态的本质是if...else，只不过实现的成绩不同。
- **重载** − 简单说，就是函数或者方法有同样的名称，但是参数列表不相同的情形，这样的同名不同参数的函数或者方法之间，互相称之为重载函数或者方法。
- **抽象性** − 抽象性是指将具有一致的数据结构（属性）和行为（操作）的对象抽象成类。一个类就是这样一种抽象，它反映了与应用有关的重要性质，而忽略其他一些无关内容。任何类的划分都是主观的，但必须与具体的应用有关。
- **封装** − 封装是指将现实世界中存在的某个客体的属性与行为绑定在一起，并放置在一个逻辑单元内。
- **反射** − 可以完整描述一个类或对象的原型。
    - 可以用来对对象进行调试，也可以获取类的信息进行hook或其他操作。

### 设计原则

- **单一职责原则**（Single Pesponsibilility Principle,SRP）
    - 避免相同的职责分配到不同的类中
    - 避免一个类承担太多指责
- **接口隔离原则**（Interface Segregation Principle,ISP）
    - 一个类对另一个类的依赖性应当是建立在最小的接口上的
    - 客户端程序不应该依赖它不需要的接口方法
- **开放 − 封闭原则**（Open–Close Principle,OCP）
    - Open – 模块的行为必须是开放的、支持扩展的，而不是僵化的
    - Closed – 在对模块的功能进行扩展时，不应该影响或者大规模影响已有的程序模块
- **替换原则**（Liskov Substitution Priciple,LSP）
    - 子类型必须能够替换掉他们的父类型、并出现在父类能够出现的任何地方
- **依赖倒置原则**（Dependence Inversion Principle,DIP）
    - 上册模块不应该依赖于下层模块，它们共同依赖于一个抽象（父类不能依赖于子类，它们都要依赖与抽象类）
    - 抽象不能依赖于具体，具体应该依赖于抽象。

## 特性、实践

### self、static、$this

- self 可以用于静态或非静态方法中，也可以用于访问类的静态属性、静态方法和常量。`self`只能引用当前类中的方法，就是说在子类中还是会调用父类的方法。

- static 可以用于静态或非静态方法中，也可以访问类的静态属性、静态方法、常量和**非静态方法**，但不能访问非静态属性。`static`
  关键字允许函数能够在运行时动态绑定类中的方法（**后期静态绑定**），帮助实现多态。

- this  **不能**存在于静态方法中，**不能**访问类的静态属性和常量。指向的是实际调用时的对象，也就是说，实际运行过程中，谁调用了类的属性或方法，$this
  指向的就是哪个对象。

个人认为静态方法本质上是面向对象过程中对面向过程编程的一种妥协，不具真正的面向对象思想。

### 魔术方法

- `__construct()`，类的构造函数——在每次创建新对象时先调用此方法
- `__destruct()`，类的析构函数——在到某个对象的所有引用都被删除或者当对象被显式销毁时执行
- `__call()`，在对象中调用一个不可访问方法时调用
- `__callStatic()`，用静态方式中调用一个不可访问方法时调用
- `__get()`，获得一个类的成员变量时调用
- `__set()`，设置一个类的成员变量时调用
- `__isset()`，当对不可访问属性调用`isset()`或`empty()`时调用
- `__unset()`，当对不可访问属性调用`unset()`时被调用。
- `__sleep()`，执行`serialize()`时，先会调用这个函数
- `__wakeup()`，执行`unserialize()`时，先会调用这个函数
- `__toString()`，类被当成字符串时的回应方法， `echo`时调用
- `__debugInfo`，用在`var_dump()`和`print_r()`时控制输出的值
- `__invoke()`，调用函数的方式调用一个对象时的回应方法
- `__set_state()`，调用`var_export()`导出类时，此静态方法会被调用。
- `__clone()`，当对象复制完成时调用

### 闭包和匿名函数

> 闭包是指在创建时封装周围状态的函数，即使闭包所在的环境的不存在了，闭包中封装的状态依然存在。
>
> 匿名函数其实就是没有名称的函数，匿名函数可以赋值给变量，还能像其他任何PHP函数对象那样传递。不过匿名函数仍然是函数，因此可以调用，还可以传入参数，适合作为函数或方法的回调。

```PHP
<?php
//使用use关键字把父作用域的变量及状态附加到PHP闭包中。
$id = $this->params['id'];
UserBankCard::destroy(function ($query) use ($id) {
  $query->where('id','=',$id);
});
//bindTo — 复制当前闭包对象，绑定指定的$this对象和类作用域。
<?php
class App
{
  protected $routes = array();
  protected $responseBody = '';
  public function addRoute($routePath,$routeCallback)
  {
    $this->routes[$roitePath] = $routeCallback->bindTo($this,__CLASS__);
  }
}
<?php
  $app = new App();
	$app->addRoute('/users/josh', function() {
    $this->responseBody = 'hello world';
  });
//把路由回调绑定到了当前的App实例上，就能在回调函数中处理App实例的状态。
```

### 流（Stream）

> 流的作用是提供统一的公共函数来处理文件、网络和数据压缩等操作。简单而言，流是具有流式行为的资源对象，也就是说，流可以线性读写，并且可以通过
> fseek() 之类的函数定位到流中的任何位置。

- http://流封装协议
    - file_get_contents( 'http://api.flickr.com/services/feeds/photos_public.gne?format=json')
- file://流封装协议
    - fopen('file:///etc/hosts', 'rb')
- php://流封装协议
    - `php://stdin`：这是个只读 PHP 流，其中的数据来自标准输入。PHP 脚本可以使用这个流接收命令行传入脚本的信息；
    - `php://stdout`：把数据写入当前的输出缓冲区，这个流只能写，无法读或寻址；
    - `php://memory`：从系统内存中读取数据，或者把数据写入系统内存。缺点是系统内存有限，所有使用 `php://temp` 更安全；
    - `php://temp`：和 `php://memory` 类似，不过，没有可用内存时，PHP 会把数据写入这个临时文件。

```php
<?php
//在日志中统计过去三十天访问某个站点的访问数据
$dateStart = new \DateTime();
$dateInterval = \DateInterval::createFromDateString('-1 day');
$datePeriod = new \DatePeriod($dateStart, $dateInterval, 30);
foreach ($datePeriod as $date) {
    $file = 'sftp://USER:PASS@rsync.net/' . $date->format('Y-m-d') . '.log.bz2';
    if (file_exists($file)) {
        $handle = fopen($file, 'rb');
        stream_filter_append($handle, 'bzip2.decompress');
        while (feof($handle) !== true) {
            $line = fgets($handle);
            if (strpos($line, 'www.example.com') !== false) {
                fwrite(STDOUT, $line);
            }
        }
        fclose($handle);
    }
}
```

```php
<?php
//自定义一个流过滤器 DirtyWordsFilter，把流数据读入内存时审查其中的脏字
class DirtyWordsFilter extends php_user_filter
{
    /**
     * @param resource $in 流入的桶队列
     * @param resource $out 流出的桶队列
     * @param int $consumed 处理的字节数
     * @param bool $closing 是否是流中最后一个桶队列
     * @return int
     * 接收、处理再转运桶中的流数据，在该方法中，我们迭代桶队列对象，把脏字替换成审查后的值
     */
    public function filter($in, $out, &$consumed, $closing)
    {
        $words = ['grime', 'dirt', 'fuck'];
        $wordData = [];
        foreach ($words as $word) {
            $replacement = array_fill(0, mb_strlen($word), '*');
            $wordData[$word] = implode('', $replacement);
        }
        $bad = array_keys($wordData);
        $good = array_values($wordData);

        // 迭代桶队列中的每个桶
        while ($bucket = stream_bucket_make_writeable($in)) {
            // 审查桶对象中的脏字
            $bucket->data = str_replace($bad, $good, $bucket->data);
            // 增加已处理的数据量
            $consumed += $bucket->datalen;
            // 把桶放入流向下游的队列中
            stream_bucket_append($out, $bucket);
        }

        return PSFS_PASS_ON;
    }
}
<?php
//然后，必须使用 stream_filter_register() 函数注册这个自定义的 DirtyWordsFilter 流过滤器
stream_filter_register('dirty_words_filter', 'DirtyWordsFilter');
//使用这个自定义的流过滤器
$handle = fopen('fuck!.txt', 'rb');
stream_filter_append($handle, 'dirty_words_filter');
while (feof($handle) !== true) {
    echo fgets($handle);
}
fclose($handle);
```

### 生成器（Generator）

> 如果要使用特定方式计算大量数据，如操作Excel表数据，对性能影响更甚。此时我们可以使用生成器，即时计算并产出后续值，不占用宝贵的内存空间

```php
<?php
//使用生成器实现
function makeRange($length) {
    for ($i=0; $i<$length; $i++) {
        yield $i;
    }
}
foreach (makeRange(1000000) as $i) {
    echo $i . PHP_EOL;
}
<?php
//导入百万行级别的数据到Excel
public static function query_use_result(string $sql)
{
  foreach (Db::query($sql) as $row )
  {
    yield $row;
  }
}
function output2excel(string $sql = null)
{
  $filename = "订单详情";//文件名
  set_time_limit(0);//不限制超时时间
  header('Content-Type: application/vnd.ms-excel;charset=utf-8');
  header('Content-Disposition: attachment;filename="' .$filename. '.csv"');
  header('Cache-Control: max-age=0');
  $fp = fopen('php://output', 'a');
  fprintf($fp, chr(0xEF).chr(0xBB).chr(0xBF));
  foreach ( static::query_use_result($sql) as $row )
  {
    fputcsv($fp, $row);//逐行写入csv文件
  }
  fclose($fp);//关闭文件
}
```

### 抽象类、接口、Trait

- 抽象类(Abstraction)
    - 抽象类，只能用于继承，而无法实例化对象。
    - 抽象类不一定含有抽象方法，但抽象方法一定存在于抽象类中。抽象方法在子类中必须被重写。
    - 只要方法没有用abstract声明，在其子类中就不用实现。而且在子类中该方法为公共方法。
    - 抽象是编程解决问题的基础，越复杂的问题，越需要一开始就对问题进行抽象，而不是直接写代码。

- 接口(Interface)
    - 接口作为一种规范和契约存在。作为规范，接口应该保证可用性；作为契约，接口应该保证可控性。
    - 接口只是一个声明，一旦使用`interface`关键字，就应该实现它。可以由程序员实现（外部接口），可以由系统实现（内部接口）。接口本身什么也不做，但是它可以告诉我们它能做什么。
    - 不足：没有契约限制；缺少足够多的内部接口。
- Trait
    - trait看上去更像是为了代码的复用而写的一个小插件，它类似于include，可以用use放在类中间，让trait里面定义的方法作为class的一部分，本身不能直接实例化
    - 从基类继承的成员会被 trait 插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了 trait 的方法，而 trait 则覆盖了被继承的方法

区别

1. 对接口的使用方式是通过关键字implements来实现的，而对于抽象类的操作是使用类继承的关键字exotends实现的，使用时要特别注意。
2. 接口没有数据成员，但是抽象类有数据成员，抽象类可以实现数据的封装。
3. 接口没有构造函数，抽象类可以有构造函数。
4. `接口中的方法都是public类型`，而抽象类中的方法可以使用private、protected或public来修饰。
5. 一个类可以同时实现多个接口，但是只能实现一个抽象类。

```php
<?php
interface 接口名称{ //使用interface关键字声明接口
常量成员 //接口中的成员属性只能是常量，不能是变量
抽象方法 function getName(); //接口中的所有方法必须是抽象方法(没有方法体)
}
?>
```

## 常用工具

### composer

#### composer.json

> Composer会使用这个文件中的信息查找、安装和自动加载PHP组件。

- name：组件的厂商名和包名
- description：简要说明组件
- keywords：几个描述组件的关键字
- homepage：组件网站的URL
- license：组件所采取的软件许可证
- authors：这是一个数组，包含每个作者的信息。
- support：组件的用户获取支持的方式
- require：列出组件自身所依赖的组件。还可以列出组件所依赖的最低PHP版本。
- require–dev：和require属性类似，不过列出的是***开发***这个组件所需的依赖。
- suggest：和require属性类似，不过只是建议安装的组件，以防与其他组合合作时需要。
- autoload：告诉Composer的自动加载器如何自动加载这个组件。在PSR–4属性中，我们要把组件的命名空间前缀与相对组件根目录的文件系统路径对应起来（命名空间要以两个反斜线结尾）。

```json
{
  "name": "qiniu/php-sdk",
  "type": "library",
  "description": "Qiniu Resource (Cloud) Storage SDK for PHP",
  "keywords": [
    "qiniu",
    "storage",
    "sdk",
    "cloud"
  ],
  "homepage": "http://developer.qiniu.com/",
  "license": "MIT",
  "authors": [
    {
      "name": "Qiniu",
      "email": "sdk@qiniu.com",
      "homepage": "http://www.qiniu.com"
    }
  ],
  "require": {
    "php": ">=5.3.3"
  },
  "require-dev": {
    "phpunit/phpunit": "~4.0",
    "squizlabs/php_codesniffer": "~2.3"
  },
  "autoload": {
    "psr-4": {
      "Qiniu\\": "src/Qiniu"
    },
    "files": [
      "src/Qiniu/functions.php"
    ]
  }
}

```

### swoole

### redis