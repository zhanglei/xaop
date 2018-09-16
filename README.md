# Xaop PHP高性能的AOP扩展 beta

### 功能特色 ###

- **基于对象的文档注解AOP模式**
- **方法注入AOP模式**(开发中)

#### 框架 ####

- **Yaf**
- **CSpeed**
- **Xannotation**

#### 系统指令及其含义 ####

- **xaop.method_prefix**

  AOP文档注解需要排除的方法前缀，也就是说如果方法以此前缀开头，那么就会跳过AOP解析

- **xaop.aop_mode**

  AOP工作模式，可选值： 1 | 2 | 3

### 安装 ###

```php
git clone https://github.com/liqiongfan/xaop.git

cd xaop

/usr/local/path_to_php/bin/phpize

./configure --with-php-config=/usr/local/path_to_php/bin/php-config
    
make -j && sudo make install
```

#### 启用对应功能扩展需要在 php.ini 文件配置指令： xaop.aop_mode，如下： ####

```ini
; To enable the AOP mode
; 1 不启用AOP
; 2 文档注解AOP模式
; 3 方法注入AOP模式
xaop.aop_mode = 2
```

#### 1、文档注解AOP模式 DEMO: ####

```php
<?php
 
/**
 *@Aspect
 */
class Swing
{
    public function _before() {
        echo '_before';
    }
    
    /**
     *@before(value="Swing._before")
     */
    public function goodLists() {
    	echo 'goodLists';
    }
}
```

因为基于 **ZendOPcode**，所以不需要使用代理对象完成切面，**直接调用方法** 即可：

```php
<?php

$swig = new Swing();
$swig->goodLists();

// 输出如下：

_before goodLists
```

文档注解支持 **自定义注解** 与扩展 **内置注解**：

- **自定义注解**

  自定义注解必须继承自 **Xaop\Annotation** 接口，并且实现 **input** 方法即可，如下示例自定义了一个 **@Tag** 注解：

```php
namespace app;

use Xaop\Annotations;

class Tag implements Annotations {

    function input($object, $annotations) {
        var_dump($object);

        foreach($annotations as $key => $val) {
            echo $key . '=' . $val . PHP_EOL;
        }
    }
}
```

使用的时候只需要传入全名即可：

```php
<?php

/**
 * @Aspect
 */
class Swing
{
    /**
     * @app\Tag( money = 5000, user = "Xaop" )
     */
    public function getMoney() {
        
    }
}
```

- **内置注解**

  内置强大的六个专用注解： **@api**、**@disable**、**@before** 、**@after**、**@success**、**@failure**

  1. **@api**

     开发 **API** 推荐使用，使用本注解，直接可以向客户端返回 **JSON** 或者 **XML** 数据，只需要在修饰的方法体返回数组数据即可，注解包含两个参数：

     **type** 与 **charset**， 如下使用：

     ```php
     /**
      *@api(type=JSON, charset=UTF-8)
      */
     public function newLists() {
     	return [ ['12' => [xxx,xxx], ['23'=>[xxx,xxx] ];
     }
     ```

     **或者**

     ```php
     /**
      *@api(type=xml, charset=UTF-8)
      */
     public function newLists() {
     	return [ ['12' => [xxx,xxx], ['23'=>[xxx,xxx] ];
     }
     ```

     **注意：参数名区分大小写，参数值不区分大小写。**

  2. **@disable**

     使用本注解可以禁用类的方法，使用本注解修饰的方法，就不会调用，并且不会提示任何错误信息，直接返回，本注解不包含任何参数。

  3. **@before**

     前置通知，在方法之前执行本注解包含的方法：如：

     ```php
     @before(value="app\models\User.startTransaction")
     ```

     使用场景：**在执行业务代码逻辑之前开启事务支持**

  4. **@after**

     后置通知，在方法之后执行本注解包含的方法：如：

     ```php
     @after(value="app\log\InvokeLog.record")
     ```

     使用场景：**在接口调用之后进行日志记录**

  5. **@success**

     方法体返回 **true** 的时候调用的通知：如：

     ```php
     @success(value="app\models\User.commit")
     ```

     使用场景：**在业务逻辑代码执行成功之后提交事务**

  6. **@failure**

     方法体返回 **false**之后调用的通知，如下：

     ```php
     @failure(value="app\models\User.rollback")
     ```

     使用场景：**在业务逻辑代码方法体返回失败的时候回滚事务**
