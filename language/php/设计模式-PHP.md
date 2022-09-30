# 设计模式

> > 《深入PHP 面向对象、模式与实践》
>
> [design-patterns-for-humans-cn](https://github.com/guanguans/design-patterns-for-humans-cn)
>
> [超全的设计模式简介（45 种）](https://learnku.com/articles/27556)
>
> [PHP笔记](https://www.kancloud.cn/a173512/php_note/1231245)

## 用于生成对象的模式

### 单例模式

> （Singleton）只生成一个对象实例的特殊类

- 此对象可以被系统中的任何对象使用
- 此对象在系统中有且应当仅有一个

```php
class DB
{
    private static $instance = null; //私有静态属性，存放该类的实例
  
 		//私有构造方法，防止在类的外部实例化
    private function __construct() {}
 
	  //私有克隆方法，防止克隆
    private function __clone() { trigger_error('Clone is not allowed !');}
 
    public static function getInstance() //公共的静态方法，实例化该类本身，只实例化一次
    {
        if (!self::$instance instanceof self) {
            self::$instance = new self;
        }
        return self::$instance;
    }
}
```

### 简单工厂模式

> （Simple Factory）又称（Static Factory）专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类

- 一个抽象产品类（可以是：接口，抽象类，普通类），可以派生出多个具体产品类。 
- 一个抽象工厂类（可以是：接口，抽象类），可以派生出多个具体工厂类。 
- 每个具体工厂类只能创建一个具体产品类的实例。 

### 工厂方法模式

> （Factory Method）构建创造者类的的继承层次

- 一个抽象产品类（可以是：接口，抽象类，普通类），可以派生出多个具体产品类。 
- 一个抽象工厂类（可以是：接口，抽象类），可以派生出多个具体工厂类。 
- 每个具体工厂类只能创建一个具体产品类的实例。 

```php
abstract class animals // 抽象产品类
{
    abstract public function animal();
}
class Cat extends animals // 具体产品类
{
    public function animal() { return "猫猫";}
}
class Dog extends animals // 具体产品类
{
    public function animal() { return "狗狗";}
}
interface Factory   // 抽象工厂类, 将对象的创建抽象成一个接口
{
    public function create();
}
class CatFactory implements Factory  // 继承工厂类, 用于实例化产品
{
    public function create() { return new Cat();}
}
class DogFactory implements Factory  // 继承工厂类, 用于实例化产品
{
    public function create() { return new Dog();}
}

class Client    // 具体操作类
{
    public function test()
    {
        $catResult = new CatFactory();
        echo $catResult->create()->animal();

        $DogResult = new DogFactory();
        echo $DogResult->create()->animal();
    }
}

$lala = new Client();
$lala->test(); //  猫猫狗狗

```

### 抽象工厂模式

> （Abstract Factory）对功能相关的产品进行分组

- 多个抽象产品类（可以是：接口，抽象类，普通类），每个抽象产品类可以派生出多个具体产品类。 
- 一个抽象工厂类（可以是：接口，抽象类），可以派生出多个具体工厂类。 
- 每个具体工厂类可以创建多个具体产品类的实例。 

```php
abstract class Cat  // 猫抽象
{
    abstract function getCat();
}
class ForeignCat extends Cat 
{
    public function getCat() { return "外国布偶猫".PHP_EOL;}
}
class ChineseCat extends Cat
{
    public function getCat() { return "华夏猫".PHP_EOL;}
}

abstract class Dog	// 狗抽象
{
    abstract function getDog();
}
class ChineseDog extends Dog
{
    public function getDog() { return "中华国犬".PHP_EOL;}
}
class ForeignDog extends Dog
{
    public function getDog() { return "外国哈士奇".PHP_EOL;}
}

interface AnimalsFactory // 抽象工厂
{
    public function createCat(); // 生产一尾猫
    public function createDog(); // 生产一头狗 -_-
}

class CreateChineseAnimalFactory implements AnimalsFactory 
{
    public function createCat()
    {
        return new ChineseCat();
    }
    public function createDog()
    {
        return new ChineseDog();
    }
}
class CreateForeignAnimalFactory implements AnimalsFactory
{
    public function createCat()
    {
        return new ForeignCat();
    }
    public function createDog()
    {
        return new ForeignDog();
    }
}

$result = new CreateForeignAnimalFactory();
$ForeignCat = $result->createCat();
echo $ForeignCat->getCat(); // 布偶猫

$ForeignDog = $result->createDog();
echo $ForeignDog->getDog(); // 哈士奇
```

### 原型模式

> （Prototype）通过克隆生成对象
>
> 用一个已经创建的实例作为原型，通过复制该原型对象来创建一个和原型相同或相似的新对象

- 多用于创建复杂的或者耗时的实例；或者创建值相等，只是命名不一样的同类数据

```php
interface Prototype { public function copy();}

class ConcretePrototype implements Prototype {
    private  $_name;
    public function __construct($name) { $this->_name = $name; } 
	  //浅copy模式，拷贝了源对象的引用地址等，原来内容变化，新内容变化
    public function copy() { return clone $this;}
  	//深copy通过序列化和反序列化完成copy，原来的内容变化，新内容不变
  	public function copy() {
      	$serialize_obj = serialize($this);
        $clone_obj = unserialize($serialize_obj);
        return $clone_obj;
    }
}

class Demo {}

// client
$demo = new Demo();
$object1 = new ConcretePrototype($demo);
$object2 = $object1->copy();
```

### 服务定位器

> （Service Locator）向系统索要对象    [Service Locator](https://xueyuanjun.com/post/2820)

```php
interface ServiceLocatorInterface
{
    public function has($interface);
    public function get($interface);
}

class ServiceLocator implements ServiceLocatorInterface
{
    private $services;
    private $instantiated;
    private $shared;

    public function __construct()
    {
        $this->services     = array();
        $this->instantiated = array();
        $this->shared       = array();
    }

    public function add($interface, $service, $share = true)
    {
        if (is_object($service) && $share) {
            $this->instantiated[$interface] = $service;
        }
        $this->services[$interface] = (is_object($service) ? get_class($service) : $service);
        $this->shared[$interface]   = $share;
    }

    public function has($interface)
    {
        return (isset($this->services[$interface]) || isset($this->instantiated[$interface]));
    }

    public function get($interface)
    {
        if (isset($this->instantiated[$interface]) && $this->shared[$interface]) {
            return $this->instantiated[$interface];
        }

        $service = $this->services[$interface];
        $object = new $service();

        if ($this->shared[$interface]) {
            $this->instantiated[$interface] = $object;
        }
        return $object;
    }
}
```

### 依赖注入

> （Dependency Injection）让系统给我们对象
>
> 可以看一下tp6.0源码分析中的依赖注入原理
>
> [Laravel 服务容器实现原理](https://learnku.com/articles/4977/laravel-service-container-implementation-princip)

- 容器就是一个对象，它知道怎样初始化并配置对象及其依赖的所有对象
- 注册会用到一个依赖关系名称和一个依赖关系的定义

```php
<?php
class Bim
{
    public function doSomething()
    {
        echo __METHOD__, '|';
    }
}

class Bar
{
    private $bim;
    public function __construct(Bim $bim)
    {
        $this->bim = $bim;
    }
    public function doSomething()
    {
        $this->bim->doSomething();
        echo __METHOD__, '|';
    }
}

class Foo
{
    private $bar;
    public function __construct(Bar $bar)
    {
        $this->bar = $bar;
    }
    public function doSomething()
    {
        $this->bar->doSomething();
        echo __METHOD__;
    }
}

class Container
{
    private $s = array();

    public function __set($k, $c)
    {
        $this->s[$k] = $c;
    }

    public function __get($k)
    {
        // return $this->s[$k]($this);
        return $this->build($this->s[$k]);
    }

    //自动绑定（Autowiring）自动解析（Automatic Resolution）
    public function build($className)
    {
        // 如果是匿名函数（Anonymous functions），也叫闭包函数（closures）
        if ($className instanceof Closure) {
            // 执行闭包函数，并将结果
            return $className($this);
        }

        /** @var ReflectionClass $reflector */
        $reflector = new ReflectionClass($className);

        // 检查类是否可实例化, 排除抽象类abstract和对象接口interface
        if (!$reflector->isInstantiable()) {
            throw new Exception("Can't instantiate this.");
        }

        /** @var ReflectionMethod $constructor 获取类的构造函数 */
        $constructor = $reflector->getConstructor();

        // 若无构造函数，直接实例化并返回
        if (is_null($constructor)) {
            return new $className;
        }

        // 取构造函数参数,通过 ReflectionParameter 数组返回参数列表
        $parameters = $constructor->getParameters();

        // 递归解析构造函数的参数
        $dependencies = $this->getDependencies($parameters);

        // 创建一个类的新实例，给出的参数将传递到类的构造函数。
        return $reflector->newInstanceArgs($dependencies);
    }

    public function getDependencies($parameters)
    {
        $dependencies = [];

        /** @var ReflectionParameter $parameter */
        foreach ($parameters as $parameter) {
            /** @var ReflectionClass $dependency */
            $dependency = $parameter->getClass();

            if (is_null($dependency)) {
                // 是变量,有默认值则设置默认值
                $dependencies[] = $this->resolveNonClass($parameter);
            } else {
                // 是一个类，递归解析
                $dependencies[] = $this->build($dependency->name);
            }
        }

        return $dependencies;
    }

    public function resolveNonClass($parameter)
    {
        // 有默认值则返回默认值
        if ($parameter->isDefaultValueAvailable()) {
            return $parameter->getDefaultValue();
        }

        throw new Exception('I have no idea what to do here.');
    }
}

// ----
$c = new Container();
$c->bar = 'Bar';
$c->foo = function ($c) {
    return new Foo($c->bar);
};
// 从容器中取得Foo
$foo = $c->foo;
$foo->doSomething(); // Bim::doSomething|Bar::doSomething|Foo::doSomething

// ----
$di = new Container();

$di->foo = 'Foo';

/** @var Foo $foo */
$foo = $di->foo;

var_dump($foo);
/*
Foo#10 (1) {
  private $bar =>
  class Bar#14 (1) {
    private $bim =>
    class Bim#16 (0) {
    }
  }
}
*/

$foo->doSomething(); // Bim::doSomething|Bar::doSomething|Foo::doSomething
```



## 使面向对象编程更加灵活的模式

### 组合模式

> （Composite） 将对象组合成树形结构以表示‘部分-整体’的层次结构，使得用户对单个对象和组合对象的使用具有一致性
>
> 个人理解：类似一个文字和一段文字都可以进行相同的操作，或是类似于文件夹系统虽是不同层级但操作方法相同。

- 灵活性：组合模式中的所有类属于同一个父类型，所以在设计中加入新的组合对象或叶子对象对外部没有影响。
- 简单性：使用组合结构的客户端只需要调用一个简单的接口即可，因为它无需知道所使用的对象是组合对象还是叶子对象。
- 隐式影响范围：组合模式中的对象以树状结构组织在一起，每个组合对象都持有指向子对象的引用。
- 现实影响范围：可以遍历树形结构来得到对象信息或对对象执行操作。

```php
<?php
abstract class Unit
{
    function getComposite() { return null; }
    abstract function bombardStrength();
}

abstract class CompositeUnit extends Unit
{
    protected $units = array();
    function getComposite()
    {
        return $this;
    }
    protected function units()
    {
        return $this->units;
    }
    function removeUnit(Unit $unit)
    {
        $this->units = array_udiff( $this->units, array($unit),
            function ($a, $b) { return ($a === $b) ? 0 : 1; }
        );
    }
    function addUnit(Unit $unit)
    {
        if (in_array($unit, $this->units, true)) { return; }
        $this->units[] = $unit;
    }
}

class Army extends CompositeUnit
{
    function bombardStrength()
    {
        $ret = 0;
        foreach ($this->units as $unit) {
            $ret += $unit->bombardStrength();
        }
        return $ret;
    }
}

class Archer extends Unit
{
    function bombardStrength()
    {
        return 4;
    }
}

class LaserCannonUnit extends Unit
{
    function bombardStrength()
    {
        return 44;
    }
}

class UnitScript
{
    static function joinExisting(Unit $newUnit, Unit $occupyingUnit)
    {
        if (!is_null($comp = $occupyingUnit->getComposite())) {
            $comp->addUnit($newUnit);
        } else {
            $comp = new Army();
            $comp->addUnit($occupyingUnit);
            $comp->addUnit($newUnit);
        }
        return $comp;
    }
}

$army1 = new Army();
$army1->addUnit(new Archer());
$army1->addUnit(new Archer());
$army2 = new Army();
$army2->addUnit(new Archer());
$army2->addUnit(new Archer());
$army2->addUnit(new LaserCannonUnit());
$composite = UnitScript::joinExisting($army2, $army1);
print_r($composite->bombardStrength());				// 60

```

> [PHP设计模式之组合模式](https://www.cnblogs.com/praglody/p/6783317.html)

```php
<?php
abstract class CompanyBase
{
    //节点名称
    protected $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }

    //增加节点
    abstract function add(CompanyBase $c);

    //删除节点
    abstract function remove(CompanyBase $c);

    //输出节点信息
    abstract function show($deep = 0);

    //节点职责
    abstract function work($deep = 0);
}

class Company extends CompanyBase
{
    protected $item = [];

    public function add(CompanyBase $c)
    {
        $nodeName = $c->getName();

        if (!isset($this->item[$nodeName])) {
            $this->item[$nodeName] = $c;
        } else {
            throw new Exception("该节点已存在,节点名称：" . $nodeName);
        }
    }

    public function remove(CompanyBase $c)
    {
        $nodeName = $c->getName();

        if (isset($this->item[$nodeName])) {
            unset($this->item[$nodeName]);
        } else {
            throw new Exception("该节点不存在,节点名称：" . $nodeName);
        }
    }

    public function show($deep = 0)
    {
        echo str_repeat("-", $deep) . $this->name . PHP_EOL;
        foreach ($this->item as $value) {
            $value->show($deep + 4);
        }
    }
    public function work($deep = 0)
    {
        foreach ($this->item as $value) {
            echo str_repeat(" ", $deep) . "[{$this->name}]" . PHP_EOL;
            $value->work($deep + 4);
        }
    }
}

class HumanResources extends CompanyBase
{

    public function add(CompanyBase $c)
    {
        throw new Exception("该节点下不能增加节点");
    }

    public function remove(CompanyBase $c)
    {

        throw new Exception("该节点下无子节点");
    }

    public function show($deep = 0)
    {
        echo str_repeat("-", $deep) . $this->name . PHP_EOL;
    }
    public function work($deep = 0)
    {

        echo str_repeat(" ", $deep) . "人力资源部门的工作是为公司招聘人才" . PHP_EOL;
    }
}

class Commerce extends CompanyBase
{

    public function add(CompanyBase $c)
    {
        throw new Exception("该节点下不能增加节点");
    }

    public function remove(CompanyBase $c)
    {

        throw new Exception("该节点下无子节点");
    }

    public function show($deep = 0)
    {
        echo str_repeat("-", $deep) . $this->name . PHP_EOL;
    }
    public function work($deep = 0)
    {
        echo str_repeat(" ", $deep) . "商务部门的工作是为公司赚取利润" . PHP_EOL;
    }
}

$c = new Company("北京某科技公司");
$h = new HumanResources("人力资源部门");
$com = new Commerce("商务部门");
$c->add($h);
$c->add($com);
//天津分公司
$c1 = new Company("天津分公司");
$c1->add($h);
$c1->add($com);
$c->add($c1);
//武汉分公司
$c2 = new Company("武汉分公司");
$c2->add($h);
$c2->add($com);
$c->add($c2);
//使用公司功能
$c->show();				北京某科技公司
                  ----人力资源部门
                  ----商务部门
                  ----天津分公司
                  --------人力资源部门
                  --------商务部门
                  ----武汉分公司
                  --------人力资源部门
                  --------商务部门
// $c->work();
```



### 装饰器模式

> （Decorator）  一种通过在运行时组合对象来拓展功能的灵活机制

```php
<?php
//装饰器接口
interface Decorator
{
    public function beforeDraw();
    public function afterDraw();
}
//具体装饰器1-宇航员装饰
class AstronautDecorator implements Decorator
{
    public function beforeDraw()
    {
        echo "穿上T恤" . PHP_EOL;
    }
    function afterDraw()
    {
        echo "穿上宇航服" . PHP_EOL;
        echo "穿戴完毕" . PHP_EOL;
    }
}
//具体装饰器2-警察装饰
class PoliceDecorator implements Decorator
{
    public function beforeDraw()
    {
        echo "穿上警服" . PHP_EOL;
    }
    function afterDraw()
    {
        echo "穿上防弹衣" . PHP_EOL;
        echo "穿戴完毕" . PHP_EOL;
    }
}
//被装饰者
class Person
{
    protected $decorators = array(); //存放装饰器
    //添加装饰器
    public function addDecorator(Decorator $decorator)
    {
        $this->decorators[] = $decorator;
    }
    //所有装饰器的前方法调用
    public function beforeDraw()
    {
        foreach ($this->decorators as $decorator) {
            $decorator->beforeDraw();
        }
    }
    //所有装饰器的后方法调用
    public function afterDraw()
    {
        $decorators = array_reverse($this->decorators);
        foreach ($decorators as $decorator) {
            $decorator->afterDraw();
        }
    }
    //装饰方法
    public function clothes()
    {
        $this->beforeDraw();
        echo "穿上长衫" . PHP_EOL;
        $this->afterDraw();
    }
}
//警察装饰
$police = new Person;
$police->addDecorator(new PoliceDecorator);
$police->clothes();
//宇航员装饰
// $astronaut = new Person;
// $astronaut->addDecorator(new AstronautDecorator);
// $astronaut->clothes();
//混搭风
// $madman = new Person;
// $madman->addDecorator(new PoliceDecorator);
// $madman->addDecorator(new AstronautDecorator);
// $madman->clothes();
```

```php
<?php
//区域抽象类
abstract class Area
{    
    abstract public function treasure();
}

//森林类，价值100
class Forest extends Area 
{
    public function treasure()
    {
        return 100;
    }
}
//沙漠类，价值10
class Desert extends Area
{
    public function treasure()
    {
        return 10;
    }
}
//区域类的装饰器类
abstract class AreaDecorateor extends Area
{
    protected $_area = null;

    public function __construct(Area $area)
    {
        $this->_area = $area;
    }
}

//被破坏了后的区域，价值只有之前的一半
class Damaged extends AreaDecorateor
{
    public function treasure()
    {
        return $this->_area->treasure() * 0.5;
    }
}

//现在我们来获取被破坏的森林类的价值
$damageForest = new Damaged(new Forest());
echo $damageForest->treasure();  //返回50
```



### 外观模式

> （Facade）  为复杂或多边的系统创建一个简单接口

```php
<?php
//医院医生员工系统
class DoctorSystem
{
    static public function getDoctor($name)
    {
        echo __CLASS__ . ":" . $name . "医生，挂你号" . PHP_EOL;
        return new Doctor($name);
    }
}
//医生类
class Doctor
{
    public $name;
    public function __construct($name)
    {
        $this->name = $name;
    }
    public function prescribe()
    {
        echo __CLASS__ . ":" . "开个处方给你" . PHP_EOL;
        return "祖传秘方，药到病除";
    }
}
//患者系统
class SufferSystem
{
    static function getData($suffer)
    {
        $data = $suffer . "资料";
        echo  __CLASS__ . ":" . $suffer . "的资料是这些" . PHP_EOL;
        return  $data;
    }
}
//医药系统
class MedicineSystem
{
    static function register($prescribe)
    {
        echo __CLASS__ . ":" . "拿到处方：" . $prescribe . "----通知药房发药了" . PHP_EOL;
        Shop::setMedicine("砒霜5千克");
    }
}
//药房
class shop
{
    static public $medicine;
    static function setMedicine($medicine)
    {
        self::$medicine = $medicine;
    }
    static function getMedicine()
    {
        echo __CLASS__ . ":" . self::$medicine . PHP_EOL;
    }
}

//通知就诊医生
$doct = DoctorSystem::getDoctor("一号医生");
//患者系统拿病历资料
$data = SufferSystem::getData("一号患者");
//医生看病历资料，开处方
$prscirbe = $doct->prescribe($data);
//医药系统登记处方
MedicineSystem::register($prscirbe);
//药房拿药
Shop::getMedicine();

echo PHP_EOL . PHP_EOL . "--------有了挂号系统以后--------" . PHP_EOL . PHP_EOL;

//挂号系统
class Facade
{
    static public function regist($suffer, $doct)
    {
        $doct = DoctorSystem::getDoctor($doct);
        //患者系统拿病历资料
        $data = SufferSystem::getData($suffer);
        //医生看病历资料，开处方
        $prscirbe = $doct->prescribe($data);
        //医药系统登记处方
        MedicineSystem::register($prscirbe);
        //药房拿药
        Shop::getMedicine();
    }
}
Facade::regist("二号医生", "二号患者");
```



## 执行及描述任务

### 解释器模式

> （Interpreter）  构造可以创建脚本应用的迷你语言解释器

```php
<?php
class Expression { //抽象表示
    function interpreter($str) { 
        return $str; 
    } 
} 

class ExpressionNum extends Expression { //表示数字
    function interpreter($str) { 
        switch($str) { 
            case "0": return "零"; 
            case "1": return "一"; 
            case "2": return "二"; 
            case "3": return "三"; 
            case "4": return "四"; 
            case "5": return "五"; 
            case "6": return "六"; 
            case "7": return "七"; 
            case "8": return "八"; 
            case "9": return "九"; 
        } 
    } 
} 

class ExpressionCharater extends Expression { //表示字符
    function interpreter($str) { 
        return strtoupper($str); 
    } 
} 

class Interpreter { //解释器
    function execute($string) { 
        $expression = null; 
        for($i = 0;$i<strlen($string);$i++) { 
            $temp = $string[$i]; 
            switch(true) { 
                case is_numeric($temp): $expression = new ExpressionNum(); break; 
                default: $expression = new ExpressionCharater(); 
            } 
            echo $expression->interpreter($temp);
            echo "<br>"; 
        } 
    } 
} 

//client
$obj = new Interpreter(); 
$obj->execute("123s45abc"); 
/* 输出：
一
二
三
S
四
五
A
B
C */
```



### 策略模式

> （Strategy）  识别系统中的算法并以它们自己特有的类型封装算法

```php
<?php
interface Strategy
{
    public function algorithmInterface();
}

class ConcreteStrategyA implements Strategy
{

    public function algorithmInterface()
    {
        echo 'algorithmInterface A' . PHP_EOL;
    }
}

class ConcreteStrategyB implements Strategy
{

    public function algorithmInterface()
    {
        echo 'algorithmInterface B' . PHP_EOL;
    }
}

class Context
{
    private $_strategy;

    public function __construct(Strategy $strategy)
    {
        $this->_strategy = $strategy;
    }

    public function contextInterface()
    {
        $this->_strategy->algorithmInterface();
    }
}

class Client
{
    static function main()
    {
        $strategyA = new ConcreteStrategyA();
        $context = new Context($strategyA);
        $context->contextInterface();

        $strategyB = new ConcreteStrategyB();
        $context = new Context($strategyB);
        $context->contextInterface();
    }
}

Client::main();
```



### 观察者模式

> （Observer）又名发布—订阅模式。  创建一种在发生系统时间时向不同对象发送通知的Hook机制。指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新

```php
<?php
abstract class Subject
{
    private $observers = array();
    public function addObserver(Observer $observer)
    {
        $this->observers[] = $observer;
        echo "添加观察者成功" . PHP_EOL;
    }
    public function delObserver(Observer $observer)
    {
        $key = array_search($observer, $this->observers); 
        if ($observer === $this->observers[$key]) { 
            unset($this->observers[$key]);
            echo '删除观察者成功' . PHP_EOL;
        } else {
            echo '观察者不存在，无需删除' . PHP_EOL;
        }
    }
    public function notifyObservers()
    {
        foreach ($this->observers as $observer) {
            $observer->update();
        }
    }
}
class Server extends Subject
{
    public function publish()
    {
        echo '今天天气很好，我发布了更新包' . PHP_EOL;
        $this->notifyObservers();
    }
}
interface Observer
{
    public function update();
}
class Web implements Observer
{
    public function update()
    {
        echo '通知已接收，web端系统更新中' . PHP_EOL;
    }
}
class App implements Observer
{
    public function update()
    {
        echo '通知已接收，APP端稍后更新' . PHP_EOL;
    }
}

$server = new Server;					
$web = new Web;
$app = new App;
$server->addObserver($web);
$server->addObserver($app);
$server->publish();
$server->delObserver($web);
$server->publish();
//添加观察者成功
//添加观察者成功
//今天天气很好，我发布了更新包
//通知已接收，web端系统更新中
//通知已接收，APP端稍后更新
//删除观察者成功
//今天天气很好，我发布了更新包
//通知已接收，APP端稍后更新
```



### 访问者模式

> （Visitor）  对对象树的所有节点应用操作，可以在不改变各元素类的前提下定义作用于这些元素的新操作

```php
<?php
interface Element
{
    public function accept(Visitor $visitor);
}

class ConcreteElementA implements Element
{
    public function accept(Visitor $visitor)
    {
        $visitor->visit($this);
    }
    public function operationA()
    {
        return "具体元素A的操作。";
    }
}

class ConcreteElementB implements Element
{
    public function accept(Visitor $visitor)
    {
        $visitor->visit($this);
    }
    public function operationB()
    {
        return "具体元素B的操作。";
    }
}

interface Visitor
{
    public function visit(Element $element);
}

class ConcreteVisitorA implements Visitor
{
    public function visit(Element $element)
    {
        if ($element instanceof ConcreteElementA) {
            print_r("具体访问者A访问-->" . $element->operationA() . PHP_EOL);
        } else if ($element instanceof ConcreteElementB) {
            print_r("具体访问者A访问-->" . $element->operationB() . PHP_EOL);
        }
    }
}

class ConcreteVisitorB implements Visitor
{
    public function visit(Element $element)
    {
        if ($element instanceof ConcreteElementA) {
            print_r("具体访问者B访问-->" . $element->operationA() . PHP_EOL);
        } else if ($element instanceof ConcreteElementB) {
            print_r("具体访问者B访问-->" . $element->operationB() . PHP_EOL);
        }

    }
}

//对象结构角色
class ObjectStructure
{
    private $list = array();
    public function accept(Visitor $visitor)
    {
        foreach ($this->list as $key => $element) {
            $element->accept($visitor);
        }
    }
    public function add(Element $element)
    {
        $this->list[] = $element;
    }

    public function remove(Element $element, $changeIndex = false)
    {
        if ($changeIndex === true) {
            $key = array_search($element, $this->list);
            if ($key !== false) {
                array_splice($this->list, $key, 1);
            }
        } else {
            $this->list[$element];
            foreach ($this->list as $key => $value) {
                if ($value === $element) {
                    unset($this->list[$key]);
                }
            }
        }
    }
}
$os = new ObjectStructure();
$os->add(new ConcreteElementA());
$os->add(new ConcreteElementB());

$visitor = new ConcreteVisitorA();
$os->accept($visitor);
$visitor = new ConcreteVisitorB();
$os->accept($visitor);
//具体访问者A访问-->具体元素A的操作。
//具体访问者A访问-->具体元素B的操作。
//具体访问者B访问-->具体元素A的操作。
//具体访问者B访问-->具体元素B的操作。
```



### 命令模式

> （Command）  创建能够保存和传递的命令对象。将请求封装成对象，从而使你可用不同的请求对客户进行参数化。

```php
<?php
abstract class Command
{
    protected $receiver;

    public function __construct(Receiver $receiver)
    {
        $this->receiver = $receiver;
    }

    abstract public function execute();
}

class ConcreteCommand extends Command
{
    public function execute()
    {
        $this->receiver->action();
    }
}

class Receiver
{
    public $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function action()
    {
        echo $this->name . '命令执行了！', PHP_EOL;
    }
}

class Invoker
{
    public $command;

    public function __construct($command)
    {
        $this->command = $command;
    }

    public function exec()
    {
        $this->command->execute();
    }
}

// 准备执行者
$receiverA = new Receiver('A');
// 准备命令
$command = new ConcreteCommand($receiverA);
// 请求者
$invoker = new Invoker($command);
$invoker->exec();
//A命令执行了！
```



### 空对象模式

> （Null Object）  用无操作的对象代替空值

```php
<?php
class Person
{
    public function code()
    {
        echo 'code makes me happy' . PHP_EOL;
    }
}

class NullObject
{
    public  function __call($method, $arg)
    {
        echo 'This is NullObject';
    }
}

function getPerson($name)
{
    if ($name == 'PHPer') {
        return new Person;
    } else {
        return new NullObject;
    }
}

$phper = getPerson('JAVAer');
$phper->code();
```

