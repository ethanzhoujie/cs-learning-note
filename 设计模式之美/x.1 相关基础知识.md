# 加餐一 | 用一篇文章带你了解专栏中用到的所有Java语法

#### 权限修饰符

权限修饰符可以修饰函数、成员变量。

- private 修饰的函数或者成员变量，只能在类内部使用。
- protected 修饰的函数或者成员变量，可以在类及其子类内使用。
- public 修饰的函数或者成员变量，可以被任意访问。

#### 继承

Java 语言使用 extends 关键字来实现继承。被继承的类叫作父类，继承类叫作子类。子类继承父类的所有非 private 属性和方法。

#### 接口

Java 语言通过 interface 关键字来定义接口。接口中只能声明方法，不能包含实现，也不能定义属性。类通过 implements 关键字来实现接口中定义的方法。

异常处理

Java 提供了异常这种出错处理机制。我们可以指直接使用 JDK 提供的现成的异常类，也可以自定义异常。在 Java 中，我们通过关键字 throw 来抛出一个异常，通过 throws 声明函数抛出异常，通过 try-catch-finally 语句来捕获异常。

```java
public class UserNotFoundException extends Exception { // 自定义一个异常
  public UserNotFoundException() {
    super();
  }

  public UserNotFoundException(String message) {
    super(message);
  }

  public UserNotFoundException(String message, Throwable e) {
    super(message, e);
  }
}

public class UserService {
  private UserRepository userRepo;
  public UserService(UseRepository userRepo) {
    this.userRepo = userRepo;
  }

  public User getUserById(long userId) throws UserNotFoundException {
    User user = userRepo.findUserById(userId);
    if (user == null) { // throw用来抛出异常
      throw new UserNotFoundException();//代码从此处返回
    }
    return user;
  }
}

public class UserController {
  private UserService userService;
  public UserController(UserService userService) {
    this.userService = userService;
  }
  
  public User getUserById(long userId) {
    User user = null;
    try { //捕获异常
      user = userService.getUserById(userId);
    } catch (UserNotFoundException e) {
      System.out.println("User not found: " + userId);
    } finally { //不管异常会不会发生，finally包裹的语句块总会被执行
      System.out.println("I am always printed.");
    }
    return user;
  }
}
```

#### package 

包Java 通过 pacakge 关键字来分门别类地组织类，通过 import 关键字来引入类或者 package。

```java
/*class DemoA*/
package com.xzg.cd; // 包名com.xzg.cd

public class DemoA {
  //...
}

/*class DemoB*/
package com.xzg.alg;

import java.util.HashMap; // Java工具包JDK中的类
import java.util.Map;
import com.xzg.cd.DemoA;

public class DemoB {
  //...
}
```



