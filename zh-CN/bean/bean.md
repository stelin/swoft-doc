# Bean

`Swoft` 中一个 `Bean` 就是一个类的一个对象实例。 容器就是一个巨大的工厂，用于存放和管理 `Bean 生命周期`。

## 创建 Bean

创建 Bean 通过 config 配置方式 还可以通过 [@Bean](#@Bean()) 注解创建 

### @Bean()
 
```php
<?php declare(strict_types=1);

namespace SwoftTest\Bean\Testing\Definition;

use Swoft\Bean\Annotation\Mapping\Bean;

/**
 * Class PrototypeClass
 *
 * @since 2.0
 *
 * @Bean(name="prototype", scope=Bean::PROTOTYPE, alias="pro")
 */
class PrototypeClass
{
    // ....
}
```
 - scope
    - Bean::SINGLETON [单例模式](./singleton.md)
    - Bean::PROTOTYPE [原型模式](./prototype.md) 
    - Bean::REQUEST  [请求](./request.md)
    - Bean::SESSION [会话](./session.md)
 - name
    指定一个为bean 指定一个名称，有时候你可能只需要配置一个 name 可以这样写`@Bean("xxx")`这样默认是参数的 name。
    
    
 - alias
    为 bean 指定一个别名, 你可以通过别名获取它
    
> `@Bean` 定义的在初始化的时候如果有`构造函数`需要注入参数 请给`默认值`，然后通过[配置](#Config)的覆盖构造函数所需参数    

> scope 注入 Bean 类型，默认单例 

### Config
```php
<?php declare(strict_types=1);


namespace SwoftTest\Bean\Testing\Definition;

/**
 * TestBean
 *
 * @since 2.0
 */
class TestBean
{

    /**
     * @var SingletonClass
     */
    private $singleton;

     /**
     * @var \JWT
     */
    private $jwt;

    /**
     * TestBean constructor.
     *
     * @param SingletonClass $singleton
     */
    public function __construct(SingletonClass $singleton)
    {
        $this->singleton = $singleton;
    }
}

```
配置方式 指定 class 的方式
```php
 'testBean'     => [
     'class'    => TestBean::class,
     [\bean('singleton')],
     'jwt'      => \bean('jwt'),
     '__option' => [
         'scope' => Bean::PROTOTYPE,
         'alias' => 'testBean-alias'
     ]
 ],
```
`testBean` 这个 key 你可以理解为是一个容器的名称。
- **class** 参数的指定 `Bean` 使用那一个类
- 下面这个数组` [\bean('singleton')],` 是`构造函数所需参数`
- **__option** 这个`swoft` 自带的你可以指定：
    - **scope** 指定 `Bean` 是用哪个级别 
    - **alias** 指定 `Bean` 的别名

当然你也可以注入`自己定义的属性` 比如：
- **jwt** 这个是上面`TestBean`类定义的属性，底层会通过`反射`注入`config`中的参数

> 通过 配置文件配置的 `Bean` 优先级`最高`因为它是最后执行的，如果配置的已经是一个 `Bean` ，`config`的 配置的将会覆盖它

## Bean 初始化

每一个 `Bean`初始化的时候会自动检查`init()`这个方法是否存在


如果存在会被首先调用，你可以在该方法中进行一些初始化操作， 作用类似 `golang` 中的`init`方法。

## 获取 Bean 
 
### @Inject
所有的 bean 都可以通过 `@Inject` 属性注入，底层会通过反射自动注入。  
```php
/**
 * @Inject("session-mg")
 *
 * @var \Session
 */
private $session;
```
- **name** 定义属性注入的bean名称。如果`name`为空，默认为`@var` 定义的类型。

### BeanFactory

获取`score`为`Bean::SINGLETON`，`Bean::PROTOTYPE`

下面 以获取 `wsRouter` 为例：
 ```php

/** @var Router $router */
$router = Swoft::getBean('wsRouter');
$router = BeanFactory::getBean('wsRouter');
$router = Container::$instance->get('wsRouter')
$router = BeanFactory::getContainer()->get('wsRouter')
```
上述这几种方式都可以`Bean::SINGLETON`，`Bean::PROTOTYPE`类型的`bean`

你也可以指定类型获取 ：

例如：获取单例类型的`bean`
```php
/* @var WsDispatcher $dispatcher */
$dispatcher = BeanFactory::getSingleton('wsDispatcher');

  
 > 注意`BeanFactory::getBean`只能获取到 框架启动加载的 bean，不能获取到 `scope` 为 `request` 和`session` 的`bean`，因为这两种类型在是在业务使用中才初始化的。
 
 [`request`](./request.md) 和 [`session`](./session.md) 的类型是 2.x 新加的
 
获取 `score`为 `request`类型的 你可以这样获取
```php
$requestBean = BeanFactory::getRequestBean(RequestClass::class, (string)Co::tid());
```
一般 `request`和 `session` 类型的 `bean` 通常是与`顶级协程ID`绑定在一起的

`score` 为 `session`的 `bean`原理是一样的只是 `session`更适合长连接会话场景，有助于你区分 `bean`类型，获取方式和`request`类型的 `bean` 大同小异
```php
$session = BeanFactory::getSessionBean(SessionClass::class, (string)Co::tid());
```
> 如果 使用`@Inject` 注入 `score` 为 `session`和`request` 类型时候，默认是与`顶级协程ID` 绑定的。


判断`当前环境`是否存在 某个 `bean`
```php
$exist = BeanFactory::hasBean($name);
```

判断是否是一个单例的 `bean`
```php
$isSingleton = BeanFactory::isSingleton('name')
```

> 不支持通过点`.`隐式调用方式获取 bean，如果需要使用`.`，指定 `bean` 的 `name` 即可，或者使用别名，具体可以参考 `db 的连接池`配置章节。