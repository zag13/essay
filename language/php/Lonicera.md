# Lonicera Framework

> 【PHP 核心技术与最佳实践】第二版 第 6 章 读书笔记  [页面来源](https://github.com/imzyf/lonicera)

Lonicera Framework - Every French soldier carries a marshal’s baton in his knapsack.

## MVC

MVC 模式的目的是实现一种动态的程序设计，使后续对程序的修改和扩展简化，并且使程序某一部分的重复利用成为可能。

## Lonicera 0.1

### bootstrap

index.php 单一入口模式。

### 路由器层

更偏向于使用 PATH_INFO 方式来访问。

从传统 URL 参数模式的访问地址进行解析，提取里面的 group、controller、action、param 4 个参数，随后交给 bootstrap 进行 dispatch
处理。

### 数据模型

用 PDO 来实现连接数据库。

- ORM Object Relational Mapping 对象与数据库的映射叫作对象关系映射
- PO Persistent Object 把一个数据库中的表的一行记录对应的对象称为持久对象
- BO Business Object 业务对象 把业务逻辑封装为一个对象
- VO Value Object 值对象 界面显示的数据对象
- DTO Data Transfer Object 用在任何需要数据传输的地方
- DAO Data Access Object 指代 Active Record 模式中的数据对象

传统的 ORM 模式提倡数据对象和负责持久化的代码的分开，但是这并没有坚持数据操作的工作量。还有一种 ORM 模式叫作 Active
Record。在 Active Record 中，模型层集成了 ORM 的功能，他们及代表实体，包含因为业务逻辑，又是数据对象，并负责把自己存储到数据库中。

Active Record 模式中的数据对象不再是 PO 对象，而是 DAO。

一系列的数据库操作组合起来，称之为 Service。Service 向下负责与数据库打交道，向上负责接收页面传递的参数以及数据的传输。理论上应该对
DAO 进项抽象到一个 Service 中。

### 视图层

PHP MVC 中的显示层开始朝着轻量化、API 化发展了。

增强一个类通常途径：

1. 使用 `__call` `__set` `__get` 等魔术方法
2. 使用反射
3. 使用 `trait`
4. 使用继承和组合

### 初步改进

`spl_autoload_register` 统一加载文件。

## Lonicera 0.2

### 引入异常机制

同时设置 `set_error_handler` 和 `set_exception_handler`。PHP 同时存在错误和异常两个互不包含的概念。

### 拦截器与插件

`dispatch` 中处理。使用正则匹配、`call_user_func`。

### Request 增强与安全防御

包装 `$_REQUEST`。

## Lonicera 0.3

### Composer 类加载机制

`PSR-4`。

### Model 增强

`illuminata/database`。

### 控制反转与依赖注入

`Inversion of Control` `IoC`，`Dependency Injection` `DI`。

- 谁控制谁？IoC 容器控制了对象。
- 控制什么？主要控制了外部资源获取（不只是对象创建，还包括比如文件等）。

传统应用程序是：由我们自己在对象中主动控制去直接获取依赖对象，而反转则是：由容器来帮忙创建及注入依赖对象。由于是容器帮我们查找及注入依赖对象，对象只是被动接受依赖对象，所以是反转。

- 哪些方面反转了？依赖对象的获取被反转了。

在我们使用 UserOrder 对象的时候，不再需要手动去创建 User 对象和 Order 对象了，而是直接问 IoC 容器去要 UserOrder 对象，IoC
容器会负责查找 UserOrder 的依赖并替我们创建 User 对象和 Order 对象，并管理它们。也就是说和我们打交道的是 IoC 容器。

依赖注入，是组件之间依赖关系由容器在运行期决定形象来说，即由容器动态地将某个依赖关系注入到组件之中。依赖注入的目的并非是为软件系统带来更多功能，而是为了提高组件重用的效率，并为系统搭建一个灵活、可扩展的平台。

通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

- 谁依赖于谁？当然是应用程序依赖于 IoC 容器。
- 为什么需要依赖？应用程序需要 IoC 容器来提供对象需要的外部资源。
- 谁注入谁？IoC 容器注入应用程序某个对象，应用程序依赖的对象。
- 注入了什么？注入某个对象所需要的外部资源(包括对象、资源、常量数据)。

IoC 和 DI 是两个相辅相成的概念，IoC 的实现是使用了 DI，而 DI 的目的是为了实现 IoC。实际上，它们是同一个概念的不同角度描述。

为什么要这么做，这么做有什么意义呢？依赖注入帮我们降低了创建对象的成本，使得对象之间松耦合。

`composer` 通过 `composer.json` 来管理第三方依赖，从这个角度讲，`composer` 就是一个 IoC 容器。

`PSR-11` `Psr\Contanier\ContainerInterface`。

## DB SQL

```mysql
SET NAMES utf8mb4;
SET
    FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`
(
    `id`   int(10) unsigned                        NOT NULL AUTO_INCREMENT,
    `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名字',
    `age`  int(10) unsigned                        NOT NULL DEFAULT '0',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;

SET
    FOREIGN_KEY_CHECKS = 1;
```
