---
layout:     post
title:      一篇夯实一个知识点系列－－python装饰器
subtitle:   
date:       2020-05-10
author:     Yao Shaomin
header-img: img/post-bg-article.jpg
catalog:    true
tags:
    - Python
---

#### 写在前面
> 本系列目的：希望可以通过一篇文章，不望鞭辟入里，但求在工程应用中得心应手。

+ 装饰器模式是鼎鼎大名的23种设计模式之一。装饰器模式可以在不改变原有代码结构的情况下，扩展代码功能。
+ Python将装饰器作为Python的一种特性，内置了对装饰器的支持，使得Python使用者在使用装饰器时更加方便，合理使用装饰器，可以使Python代码极具美感。
+ 由于设计模式是一套被反复使用的代码设计经验，并不是编码必备的技能。所以在编码过程中，完全放弃使用装饰器。但是如果你不写出pythonic风格的，没有坏味道的代码，那么装饰器是这条路上绕不过的坎儿。

#### 干货儿
> 包含八节内容：闭包(实现装饰器的基础)，不带参数的函数装饰器，带参数的函数装饰器，不带参数的类装饰器，带参数的类装饰器，常用内建装饰器，装饰器总结(套路总结)，装饰器经典实例(单例模式)。

+ 闭包

    > 装饰器是通过闭包实现的。闭包是一个比较复杂的话题，深了说可以讲到python对常量表和符号表的处理方式。这里只做简单介绍。个人认为只要记住以下三个特性，就明白了闭包的概念。

    - 一个闭包是一个作用域。一个闭包只能访问作用域内的local变量和作用域外的nonlocal变量。
    - 如果在作用域外有和作用域内同名的变量var，如果在作用域内先使用变量var,然后再定义变量var，那么会抛出a变量先使用后定义的错误。
    - 闭包可以将作用域"封装"。那么我们可以在闭包之外，访问闭包内的局部变量。因为局部变量被"封装"在了闭包内。

- 不带参数函数装饰器
    假设有一个需求，我们需要在每个函数运行时,打印当下时刻的时间戳。那么有以下两种写法：

    * 不使用装饰器
        编写一个打印时间戳的工具函数，编写一个业务函数。传入业务函数对象到工具函数中，实现打印时间戳并执行业务函数的需求。代码如下：

        ```python
        import time

        def f():
            print("f is running!")

        def f1():
            print("f1 is running!")

        def print_running_time(f):
            print("running time:", time.time())
            f()

        print_running_time(f)
        print_running_time(f1)

        >>> ('running time:', 1588864281.154459)
            f is running!
            ('running time:', 1588864281.154483)
            f1 is running!
        ```

    - 使用装饰器(函数装饰器)
    
        编写一个打印时间戳的装饰器函数，编写一个业务函数。装饰器函数装饰业务函数，实现打印时间戳并执行业务函数的需求。代码如下：

        ```python
        import time

        def print_running_time(f):
            def wrapper():
                print("running time:", time.time())
                f()
            return wrapper

        @print_running_time
        def f():
            print("f is running!")

        @print_running_time
        def f1():
            print("f1 is running!")

        f()
        f1()

        >>> ('running time:', 1588864281.154459)
                 f is running!
                ('running time:', 1588864281.154483)
                f1 is running!
        ```
      
    - 以上两种写法对比
        通过对比以上两种写法，我们可以发现最明显的区别是代码在运行时，第一种写法执行的print_running_time函数，第二种写法执行的是f函数。那么明显第二种写法中抽象出的语义更加接近我们的业务需求。在同样需求增加的情况下，第一种写法需要写更多的工具函数，并且在执行业务函数时需要进行多层嵌套，极大地增加了代码的复杂度。第二种写法可以增加多个装饰函数装饰到业务函数上方，在多需求下依旧保持代码的可读性和层次感，功能的独立性和扩展性。
        - 初探装饰器原理
    
            装饰器的代码运行分为两步，装饰器初始化(在运行至被装饰函数定义处)和执行被装饰函数(在运行至被装饰函数调用处)
            以第二种写法装饰器的写法为例，装饰器的原理如下：
    
            - 在代码加载过程中，代码从上往下执行，那么在执行到#1代码时，相当于执行了#2代码。(#1和＃2的代码是等价的。＠docorator_func装饰ｆ，就相当于执行decorator_func(f))。根据#2代码中print_running_time可知，执行print_running_time(f)的返回值是wrapper(注意返回的是函数对象wrapper，不是wrapper()).
            
                ```python
                # 1
                @print_running_time
                def f():
                    print("f is running!")
        
                # 2
                print_running_time(f)
        
                # 3
                def print_running_time(f):          #3.1
                    def wrapper():                  #3.2
                        print("running time:", time.time())     #3.3
                        f()                                     #3.4
                    return wrapper
        
                # 4
                f()
             
                ```
        
            - 那么源码中#1处的三行代码，返回值为wrapper，即相当于通过增加@装饰函数，f现在已经指向了wrapper对象。
            - 根据之前提到闭包的特性：闭包可以访问作用之外的非局部变量，可以将作用域"封装",在闭包之外访问闭包内的变量。所以wrapper可以访问＃3.1中到自己外层函数的参数ｆ变量(被装饰器函数对象)，并且可以封装wrapper作用域，保存ｆ变量。
            - 执行#4处的业务函数f(),即执行#3.2的wrapper()代码，即执行#3.3和#3.4代码。
            - 整个过程中需要注意的是，在代码运行至#1时，f作为装饰器参数被#3.2wrapper闭包保留，在#1执行完之后，会存在两个f对象，#4的f对象指向wrapper，#3.4的f对象依旧是#1处的f对象。
            - 执行流程为f()==> wrapper()==> 执行#7.1 #7.2代码==>打印当前时刻时间戳，顺利执行了原有的业务函数。

- 带参数的函数装饰器

    现在有新的需求，根据调试和生产环境的不同，需要往复地开关打印时间戳的功能，那么这时就需要为装饰器函数增加参数，来作为是否打印时间戳的开关。如以下代码所示，ｆ()会打印当前函数的执行时间，ｆ1()则不会打印函数的执行时间
    
   ```python
        import time
    
        ＃ １
        def print_running_time(*flag):
            def outer_wrapper(f):
                def inner_wrapper():
                    if flag:
                        print("running time:", time.time())
                    f()
                return inner_wrapper
            return outer_wrapper
    
        ＃ ２
        @print_running_time(1)
        def f():
            print("f is running!")
    
        # 3
        @print_running_time()
        def f1():
            print("f1 is running!")
    
        f()
        f1()
    
        >>> ('running time:', 1588860065.265516)
            f is running!
            f1 is running!
   ```
   
  
    带参数装饰器原理

    - 之前简单装饰器原理==>＠decorator_func装饰业务函数ｆ<=>decorator_func(f),那么＃２处的代码<=>print_running_time(1)(f)<=>outer_wrapper(f)<=>inner_wrapper。需要注意的是inner_wrapper作为闭包，包含了外层两个变量flag和f的原始值。
    - 接下来调用f(),执行inner_wrapper(),通过判断flag真假，选择是否打印当前时间戳，然后执行业务函数，实现需求。

- 不带参数类装饰器
    - 准确来说，装饰器的本质是将一个可调用对象作为参数传入另一个可调用对象，然后通过闭包保存变量，在适当的时候执行。我们知道，python有两个特性

        + Python中函数和类都是一等对象(这也是装饰器能作为python特性的原因之一)。
        + python中若callable(obj)为真，那么这个对象就是可调用的。所以类，函数，方法，实现了__call__魔术方法的类实例，都是可调用对象。
        
    - 根据装饰器的本质和以上Python两个特性可以得出以下结论：

        * 函数和类都可以作为装饰器，也可以被装饰器装饰。
        * 类装饰器和函数装饰器思路相同，__init__作为对象初始化的第一步,可以实现一层闭包的效果

    - 将之前简单函数装饰器的例子换成类装饰器，代码如下(为了与之前代码保持一致，所以类名不符合Python命名规范)：
      
        ```python
        import time

        class print_running_time:
            def __init__(self, f):  # 相当于闭包，通过实例属性保存变量f实现闭包中的变量封装
                self.f = f

            def __call__(self):     # 类实例可以被调用 
                print("running time:", time.time())
                return self.f()

        @print_running_time
        def f():
            print("f is running!")

        @print_running_time
        def f1():
            print("f1 is running!")

        f()
        f1()

        >>> 输出同简单函数装饰器
        ```

- 带参数的类装饰器

    - 将之前简单函数装饰器的例子换成类装饰器，代码如下：

        ```python
        import time
    
        class print_running_time:
            def __init__(self, *flag):  # 相当于闭包，通过实例属性保存变量flag实现闭包中的变量封装
                self.flag = flag
    
            def __call__(self, f):     # 类实例可以被调用,传入业务函数f
                def wrapper():
                    if self.flag:
                        print("running time:", time.time())
                    f()
                return wrapper   
    
        @print_running_time(1)
        def f():
            print("f is running!")
        
        @print_running_time()
        def f1():
            print("f1 is running!")
        
        f()
        f1()
        
        >>> 输出同带参数的函数装饰器
      ```
  
- 常用内建装饰器
    > 装饰器是Python最重要的特性之一，Python实现了很多对装饰器的支持

    - wraps
       wraps可以保留被装饰函数的__doc__。如下代码所示，wraps装饰器的开关会导致打印f.\_\_doc__出现两种结果

        + 如果注释掉#1.1的代码，打印结果为#1.3
        + 如果加上#1.1的代码，打印结果为#2.1
       
        ```python
        import time
        from functools import wraps

        # 1
        def print_running_time(f):
            @wraps(f)                   # 1.1
            def wrapper():
                '''the func wrapper'''  # 1.3
                print("running time:", time.time())
                f()
            return wrapper

        # 2
        @print_running_time
        def f():
            '''the func f'''        # 2.1
            print("f is running!")

        print(f.__doc__)

        >>> the func f
        ```
    - property 、setter、 deleter
        > 这三个是孪生兄弟，其中property用的最多，setter和deleter依附property。
             
             - property:将函数调用转化为属性
             - setter:设置属性
             - deleter:删除属性
        - 类似于JavaBean，可以将对属性的操作写入函数中，限制属性操作，保护
        属性安全。代码如下：

        ```python
        class Student(object):
        
            @property
            def name(self):
                return self._name
        
            @name.setter
            def name(self, name):
                if len(name) < 2:
                    raise ValueError("无名大侠？")
                self._name = name
        
            @name.deleter
            def name(self):
                del self._name
        
        stu = Student()
        stu.name = "刘"      # name.setter
        print(stu.name)         ## property
        del stu.name           # name.deleter
        print(stu.name)         # raise AttributeError  
        
        ```
- 多装饰器叠加

    多个装饰器叠加是python中很常见的骚操作，如Flask和Django中都会用到，举例如下：

    ```python
    import sys

    # 1
    def f1(func):
        print('f1 start')
        def wrapper():              # 1.1
            print('f1 ' + sys._getframe().f_code.co_name + ' start')
            func()                  # 1.2
            print('f1 ' + sys._getframe().f_code.co_name + ' end')
        print('f1 end')
        return wrapper
    
    # 2 
    def f2(func):
        print('f2 start')
        def wrapper():                  # 2.1
            print('f2 ' + sys._getframe().f_code.co_name + ' start')
            func()                      # 2.2
            print('f2 ' + sys._getframe().f_code.co_name + ' end')
        print('f2 end')
        return wrapper
    
    # 3
    @f1
    @f2
    def func():                 #3.1
        print('the func')
    
    #4
    func()                      #4.1              

    >>> f2 start
            f2 end
            f1 start
            f1 end
            f1wrapper start
            f2wrapper start
            the func
            f2wrapper end
            f1wrapper end
    ```
    - 多装饰器执行过程分析
    执行分为两步，装饰器初始化，被装饰函数执行。顺序如下：
   1. 装饰器初始化，根据装饰器原理，#3处的代码等价于f1(f2(func))
   1. 执行f2(func); >>> f2 start  f2 end; return #2.1处的wrapper(#2.2处的func为#3.1处的func)
   2. 执行f1(f2(func))==> f1(#2.1处的wrapper); >>> f1 start  f1 end; return #1.1处的wrapper(#1.2处的func为#2.1处的wrapper)
   3. 装饰器初始化结束, 以上两步的输出如下：
       ```python
       >>> f2 start
               f2 end
               f1 start
               f1 end
       ```
   4. 被装饰函数执行：func() <=> #1.1处的wrapper，替换之后代码如下
       ```python
       print('f1 ' + sys._getframe().f_code.co_name + ' start')
       func()                  # 1.2
       print('f1 ' + sys._getframe().f_code.co_name + ' end')
       ```
       \#1.2处的func <=> # 2.1处的wrapper，替换之后代码如下
       ```python
       print('f1 ' + sys._getframe().f_code.co_name + ' start')
       print('f2 ' + sys._getframe().f_code.co_name + ' start')
       func()                      # 2.2
       print('f2 ' + sys._getframe().f_code.co_name + ' end')
       print('f1 ' + sys._getframe().f_code.co_name + ' end')
       ```
       \#2.2处的func即为#4.1处的func，执行以上代码，输出结果如下：
        ```python
         >>> f1wrapper start
             f2wrapper start
             the func
             f2wrapper end
             f1wrapper end
        ```


- 装饰器总结
    -  装饰器原理：#1代码与#2代码等价
        ```python
            # 1
            @decorator_func
            def func():
                pass
    
            # 2
            decorator_func(func)
        ```
    - 装饰器套路
            不带参数的函数装饰器需要有两层函数：
        - 外层函数参数为被装饰函数对象
        - 内层参数为被装饰函数的参数
    
          带参数的函数装饰器需要有三层函数：
          - 外层函数参数为装饰器函数参数(简直是废话，外层函数本来就是装饰器函数)
          - 中层函数参数为被装饰函数对象
          - 内层参数为被装饰函数的参数
    
          类装饰器同理，最外层函数可以用__init_函数代替，中层(如果有和内层函数写在__call__中
    - 多个装饰器叠加
      根据业务函数和装饰器函数的距离，由近及远执行装饰器函数(外层函数)，然后由远到近执行内层函数。

- 装饰器经典实例：单例模式
    > 以下均单进程可行，多线程需要加锁

    单例模式

    ```python
    # eg:1
    class Singleton:
        _singleton = None

        def __new__(cls):
            if cls._singleton is None:
                cls._singleton = super().__new__(cls)
            return cls._singleton

    ins1 = Singleton()
    ins2 = Singleton()
    print(ins1 is ins2)
    
    # eg:2
    def singleton(cls):
        ins_pool = {}
    
        def inner():
            if cls not in ins_pool:
                ins_pool[cls] = cls()
            return ins_pool[cls]
        return inner
    
    @singleton
    class Cls:
        def __init__(self):
            pass
    
    ins1 = Cls()
    ins2 = Cls()
    print(ins1 is ins2)
    
    # eg:3
    class Singleton:
    
        def __init__(self, cls):
            self.ins_pool = {}
            self.cls = cls
    
        def __call__(self):
            print(self.ins_pool)
            if self.cls not in self.ins_pool:
                self.ins_pool[self.cls] = self.cls()
            return self.ins_pool[self.cls]

    @Singleton
    class Cls:
        def __init__(self):
            pass
    
    ins1 = Cls()
    ins2 = Cls()
    print(ins1 is ins2)
    ```

#### 写在最后
 希望大家可以通过本文掌握装饰器这个杀手级特性。欢迎关注个人博客:[药少敏的博客](https://shaominyao.github.io)