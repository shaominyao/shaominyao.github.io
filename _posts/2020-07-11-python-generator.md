---

layout:     post
title:      一篇夯实一个知识点系列－－python生成器
subtitle:   
date:       2020-07-11
author:     Yao Shaomin
header-img: img/post-bg-article.jpg
catalog:    true
tags:
-   Python
---
#### 写在前面

> 本系列目的：一篇文章，不求鞭辟入里，但使得心应手。

-   迭代是数据处理的基石，在扫描内存无法装载的数据集时，我们需要一种惰性获取数据的能力(即一次获取一部分数据到内存)。在Python中，具有这种能力的对象就是迭代器。生成器是迭代器的一种特殊表现形式。

 - 个人认为生成器是Python中最有用的高级特性之一(甚至没有之一)。虽然初级编码中使用寥寥，但随着学习深入，会发现生成器是协程，异步等高级知识的基石。Python最有野心的asyncio库，就是用协程砌造的。

    >   注：生成器和协程本质相同。PEP342(Python增强提案)增加了生成器的send()方法，使其变身为协程。如此之后，生成器生成数据，协程消费数据。虽然本质相同，但是由于从理念上说协程跟迭代没有关系，并且纠缠生成器和协程的区别与联系会引爆自己的大脑，所以应该将这两个概念区分。此处说本质相同意为：理解生成器原理之后，理解增加了send方法，但是实现方式几乎相同的协程会更加轻松(这段话看不懂没有关系，船到桥头自然直，学到协程自然懂)。

-   Python的一致性是其最迷人的地方。了解了Python生成器，迭代器的实现。就会对Python的一致性设计有更加强烈的感知。本文读完之后，遇到面试官提问为什么列表可以迭代，字典可以迭代，甚至文本文件都可以迭代时，你就可以稳(huang)得一批。

-   阅读本文之前，如果你对Python的一致性有一些了解，如鸭子类型，或者Cpython的PyObject结构体，那真是太棒了。不过鉴于笔者深厚的文字功底，没有这些知识也不打紧。

#### 干货儿

-   迭代器

    >   在学习生成器之前，先要了解迭代器。顾名思义，迭代器即具有迭代功能的对象。在Python中，可以认为迭代器可以通过不断迭代，产生出一个又一个的对象。

    -   可迭代对象和迭代器

        >   Python的一致性是靠协议支撑的。一个对象只要遵循以下协议，它就是一个可迭代对象或迭代器。

        -   Python中的一个对象，如果实现了iter方法，并且iter方法返回一个迭代器，那么它就是可迭代对象。如果实现了iter和next方法，并且iter方法返回一个迭代器，那么它就是迭代器(有点绕，按住不表，继续学习)。

            >   注：如果对象实现了\_\_getitem\_\_方法，并且索引从0开始，那么也是可迭代对象。此hack为兼容性考虑。只需切记，如果你要实现可迭代对象和可迭代器，那么请遵循以上协议。

        -   可迭代对象的iter返回迭代器，迭代器的iter方法返回自身(也是迭代器)，迭代器的next方法实现迭代功能，不断返回下一个元素，或者在元素为空时raise一个StopIteration终止迭代。

    -   可迭代对象与迭代器的关系

        话不多说，上代码。

        ```python
        class Iterable:
            def __init__(self, *args):
                self.items = args
        
            def __iter__(self):
                return Iterator(self.items)       
        
        class Iterator:
            def __init__(self, items):
                self.items = items
                self.index = 0
        
            def __iter__(self):
                return self                       
        
            def __next__(self):                
                try:
                    item = self.items[self.index]
                except IndexError:
                    raise StopIteration()
                self.index += 1
                return item
        
        ins = Iterable(1,2,3,4,5)        # 1
        for i in ins:
            print(i)
        print('the end...')
        >>> 											 # 2
        1
        2
        3
        4
        5
        the end ...
        ```

        -   上述代码中，实现了可迭代对象Iterable和迭代器Iterator。遵循协议规定，Iterable实现了iter方法，且iter方法返回迭代器Iterator实例，迭代器实现了iter方法和next方法，iter返回自身(即sel，迭代器本身f)，next方法返回迭代器中的元素或者引发StopIteration异常。运行上述代码，会看到#2处的输出。

        -   通过上述代码迭代一个对象显得十分啰嗦。比如在Iterable中，iter必须要返回一个迭代器。为什么不能直接用Iterator迭代元素呢？假设我们通过迭代器来迭代元素，将上述代码中的\#1处如下代码：

            ```python
            ins = Iterator([1,2,3,4,5])
            for i in ins:														# 3
                print(i)
            for i in ins:														# 4
                print(i)
            next(ins)															# 5
            print('the end...')
            >>>  																	# 6
            1
            2
            3
            4
            5
            ...
            File "/home/disk/test/a.py", line 20, in __next__		# 7
                raise StopIteration()
            the end...
            ```

            运行上述代码，会看到#6处的输出。疑惑的是，#3和#4处运行了两次for循环，结果只打印一遍所有元素。解释如下：

            -   上述代码中，ins是一个Iterator迭代器对象。那么ins符合迭代器协议：每次调用next，会返回下一个元素，直到迭代器元素为空，raise一个StopIteration异常。

            -   \#3处第一次通过for循环迭代ins，相当于不断调用ins的next方法，不断返回下一个元素，输出如#6所示。当元素为空时，迭代器raise了StopIterator。而这个异常会被for循环捕获，不会暴露给用户，所以我们就认为数据迭代完成，并且没有出现异常。

            -   迭代器ins内的元素已经被#3处的for循环消耗完，并且raise了StopIteration(只不过被for循环捕获静默处理，没有暴露给用户)。此时ins已经是元素消耗殆尽的“空”状态。在#4处第二次通过for循环迭代ins，因为ins内的元素为空，继续调用ins的next方法，那么还是会raise一个StopIteration，而且又被for循环静默处理，所以没有异常，也没有输出。

            -   接下来，#5处通过next方法获取ins的下一个元素，同上，继续raise一个StopIteration异常。由于此处通过next调用而不是for循环，异常不会被处理，所以抛出到用户层面，即#7输出。

            -   重新编写上述代码中#3处for循环和#4处for循环，可以看到对应输出验证了我们的结论。第一次for循环在迭代到元素为2时跳出循环，第二次for循环继续迭代同一个迭代器，那么会继续上次迭代器结束位置继续迭代元素。代码如下：

                ```Python
                ins = Iterator([1,2,3,4,5])
                print('the first for:')
                for i in ins:								    # 3  the first for
                  print(i)
                  if i == 2:
                      break
                print('the second for:')
                for i in ins:								   # 4 the second for
                      print(i)
                print('the end...')
                >>>												# the output
                the first for:
                1
                2
                the second for:
                3
                4
                5
                the end...
                ```

                所以我们可以得到如下结论：

                -   一个迭代器对象只能迭代一遍。多次迭代，相当于不停对一个空迭代器调用next方法，会不停raise StopIteration异常。
                -   由于迭代器实现了iter方法，并且iter方法返回了迭代器，那么迭代器也是一个可迭代对象(废话，不是可迭代对象，上述代码中如何可以用for循环迭代呢)
                -   **综上来说，可迭代对象和迭代器明显是一个多态的问题。迭代器是一个可迭代对象，可以迭代返回元素，由于iter返回self(即自身实例)，所以只能迭代一遍，迭代到末尾就会抛出异常。而每次迭代可迭代对象，iter都会返回一个新的迭代器实例。所以可迭代对象是支持多次迭代的。比如l=[i for i in range(10)]生成的list对象就是一个可迭代对象，可以被多次迭代。l=(i for i in range(10))生成的是一个迭代器，只能被迭代一遍。**

    -   迭代器支持

        >   引用流畅的Python中的原话，迭代器支持以下6个功能。由于篇幅所限，点到为止。大家只要理解了迭代器的原理，理解以下功能自然是水到渠成。

        -   for循环

            上述代码已经有举例，可参考

        -   构建和扩展集合类型

            ```python
            from collections improt abc
            
            class NewIterator(abc.Iterator):
                pass													# 放飞自我，实现新的类型
            ```

        -   列表推导，字典推导和集合推导

            ```python
            l = [i for i in range(10)]			# list
            d = {i:i for i in range(10)}	  # dict
            s = {i for i in range(10)}		   # set
            ```

        -   遍历文本文件

            ```python
            with open ('a.txt') as f:
                for line in f:
                    print(line)
            ```

        -   元祖拆包

            ```python
            for i, j in [(1, 2), (3, 4)]:
                print(i,  j)
            >>>
            1 2
            3 4
            ```

        -   调用函数时，使用*拆包实参

            ```python
            def func(a, b, c):
                print(a, b, c)
            
            func(*[1, 2, 3])  # 会将[1, 2, 3]这个list拆开成三个实参，对应a, b, c三个形参传给func函数
            ```

-   生成器

    >   Python之禅曾经说过，simple  is better than complex。鉴于以上代码中迭代器复杂的实现方式。Python提供了一个更加pythonic的实现方式——生成器。生成器函数就是含有yield关键字的函数(目前这种说法是正确的，之后会学到yield from等句法，那么这个说法就就需要更正了)，生成器对象就是调用生成器函数返回的对象。

    -   生成器的实现

        >   将上述代码修改为生成器实现，如下：

        ```python
        class Iterable:
            def __init__(self, *args):
                self.items = args
        
            def __iter__(self):							# 8
                for item in self.items:
                    yield item
        
        ins = Iterable(1, 2, 3, 4, 5)
        print('the first for')
        for i in ins:
            print(i)
        print('the second for')
        for i in ins:
            print(i)
        print('the end...')
        
        >>>															# 9							
        the first for
        1
        2
        3
        4
        5
        the second for
        1
        2
        3
        4
        5
        the end...
        ```

        上述代码中，可迭代对象的iter方法并没有只用了短短数行，就完成了之前Iterator迭代器功能，点赞！

    -   yield关键字

        要理解以上代码，就需要理解yield关键字，先来看以下最简单的生成器函数实现

        ```Python
        def func():
            yield 1																
            yield 2
            yield 3
        
        ins1 = func()
        ins2 = func()
        print(func)
        print(ins1)
        print(ins2)
        
        for i in ins1:
            print(i)
        for i in ins1:
            print(i)
        
        print(next(ins2))
        print(next(ins2))
        print(next(ins2))
        print(next(ins2))
        
        >>> 
        <function func at 0x7fcb1e4bde18>
        <generator object func at 0x7fcb1cc7c0a0>
        <generator object func at 0x7fcb1cc7c0f8>
        1
        2
        3
        1
        2
        3
          File "/home/disk/test/a.py", line 18, in <module>
            print(next(ins2))
        StopIteration
        ```

        从以上代码可以看出：

        -   func是一个函数，但是调用func会返回一个生成器对象，并且通过打印的地址看，每次调用生成器函数会返回一个新的生成器对象。
        -   生成器对象和迭代器对象相似，都可以被for循环迭代，都只能被迭代一遍，通过next调用，都会在生成器元素为空时raise一个StopIteration异常。

        那么含有yield关键字的生成器函数体是如何执行的呢？请看如下代码：
        ```Python
        def f_gen():							# 10
            print('start')
            yield 1									# 11
            print('stop')
            yield 2									# 12
            print('next')
            yield 3									# 13
            print('end')
        
        for i in f_gen():					# 14
            print(i)
        
        >>>
        start
        1
        stop
        2
        next
        3
        end
        ```
        从上述代码及其打印结果，我们可以得出如下结论：

        -   \#10处代码表明，生成器函数定义与普通函数无二，只是需要包含有yield关键字
        -   \#14for 循环隐形调用next的时候，会执行到#11处，打印start，然后产出值 1返回给for循环，打印
        -   for 循环继续调用next，**从#11处执行到#12处**#，打印stop，然后产出值 2返回给for循环，打印
        -   for 循环继续调用next，**从#12处执行到#13处**#，打印next，然后产出值 3返回给for循环，打印
        -   for 循环继续调用next，**从#13处执行到函数尾**#，打印end，然后raise一个StopIteration，由于for循环捕获异常，程序正常执行
        -   **综上所述，yield具有暂停的功能，每次迭代生成器，生成器函数体都会前进到yield语句处，并将yield之后的值抛出(无值抛None)。生成器函数作为一个工厂函数，实现了可迭代对象中iter函数的功能，可以每次产出一个新的迭代器实例。由于使用了特殊的yield关键字，它拥有与区别于迭代器的新名字——生成器，它其实与迭代器并无二致**

-   生成器表达式

    >   将列表推导式中的[]改为()，即为生成器表达式。返回的是一个生成器对象。一般用户列表推导但是又不需要立马产生所有值的情景中。

    ```python
    gen = (i for i in range(10))
    
    for i in gen:
        print(i)
    
    for i in gen:						# 只能被消费一遍，第二遍无输出
        print(i)
    print('the end...')
    
    >>> 
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9
    the end...
    ```

-   itertools

    >   python的内置模块itertools提供了对生成器的诸多支持。这里列举一个，其它支持请看文档

    ```Python
    gen = itertools.count(1, 2)    # 从1开始，步长为2，不断产生数值
    
    >>> next(gen)
    1
    >>> next(gen)
    3
    >>> next(gen)
    5
    >>> next(gen)
    7
    >>> next(gen)
    9
    >>> next(gen)
    11
    ```

-   yield from 关键字

    >   yield from 是python3.3中出现的新句法。yield from句法可以实现委派生成器。

    ```Python
    def func():
        yield from (i for i in range(5))
    
    gen = func()
    
    for i in gen:
        print(i)
        
    >>>
    0
    1
    2
    3
    4
    ```

    如上所示，yield from把func作为了一个委派生成器。for循环可以通过委派生成器func直接迭代子生成器(i for i in range(5))。不过只是这个取巧远远不足以将yield from作为一个新句法加入到Python中。比起上述代码的迭代内层循环，新句法更加重要的功能是委派生成器为调用者和子生成器建立了一个管道。通过生成器的send方法就可以在管道中为两端传递消息。如果使用此方法在程序层面控制线程行为，就会迸发出强大的能量，它叫做协程。

#### 写在最后

---

-   注意事项

    >   迭代器与生成器功能强大，不过使用中还是有几点要注意：

     -   迭代器应该实现iter方法，虽然很多时候不实现此方法页不会影响代码运行。实现此方法的最主要原因有二：
        -   迭代器协议规定需要实现此方法
        -   可以通过issubclass检查对象是否是迭代器
    -   不要把可迭代对象变为迭代器。原因有二：
        -   这不符合迭代器协议规定，造就了一个四不像。
        -   可迭代对象应该是可以重复遍历的，如果变为了迭代器，那么只能遍历一次。

-   tips

    >   个人觉得迭代器有趣的点

    -   os.walk

        os.walk迭代器可以深度遍历目录，是个大杀器，你值得拥有，快去试试吧。

    -   iter

        iter可以接受两个位置参数：callable和flag。callable()可以不断产出值，如果等于flag，则终止。如下是一个小例子

        ```python
        gen = (i for i in range(10))
        for i in iter(lambda: next(gen), 4):				# 执行ntext(gen)， 不断返回生成器中的值，等于4则停止
            print(i)
        
        >>> 
        0
        1
        2
        3
        the end...
        ```
    
    -   yield可以接收值
    
        yield可以接收send发送的值。如下代码中，#16处send的值，会传给#15中的yield，然后赋值给res。
    
        ```python
        def func():
            res = yield 1				#15
            print(res)			
        
        f = func()
        f.send(None)			  # 预激
        f.send(5)						# 16
        ```
    
----

希望大家可以通过本文掌握装饰器这个杀手级特性。欢迎关注个人博客:[药少敏的博客](https://shaominyao.github.io)

