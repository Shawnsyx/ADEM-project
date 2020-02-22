&emsp;多学了python后对python当中的下划线很感兴趣。不同于c或者java中局限于“命名作用”作用的下划线，python中的下划线在表示特殊含义方面上有着重要作用。

&emsp;在创建类的时候，也会较易发现`def __init__():`，只要有自动补全的IDE，就会默认出现下划线，前后都有。而不同的函数下划线长度还不同（长度为1或为2，前后出现表意也不同）。这边引起了我的好奇。

参考文章：[《Python中下划线的5种含义》](https://blog.csdn.net/tcx1992/article/details/80105645)和[*The Meaning of Underscores in Python*](https://dbader.org/blog/meaning-of-underscores-in-python)

- 单前导下划线: _var
- 单末尾下划线：var_
- 双前导下划线：__var
- 双前导和末尾下划线：__var__
- 单下划线：_

# 单前导下划线 _var
&emsp;Intended for internal use. [It is defined there.](http://pep8.org/#descriptive-naming-styles) 但是python并不强制要求，而且不像Java，Python对“公共”和“私有”的区分并不那么明显。就像一种提示：
> 嘿，这其实不是这个类的一个公共接口，最好别管它。
举例如下：
```Python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23
```

实例化之后会发生啥呢？

```Python
>>> t = Test()
>>> t.foo
11
>>> t._bar
23
```

&emsp;所以单前导下划线并没有阻止我们访问到方法`_bar`
&emsp;但是如果使用通配符从模块中导入名称的话，python不会导入带有单前导下划线名称的方法。（我不确定在python中叫“方法”是否合适，欢迎指摘。）

比如，命名一个`my_modelue`有一下代码：
```Python
# This is my_module.py:
 
def external_func():
   return 23
 
def _internal_func():
   return 42
```

此时，如果用通配符*导入所有名称，则Python不会导入有单前导下划线的名称（除非模块定义了覆盖此行为的__all__列表）
```Python
>>> from my_module import *
>>> external_func()
23
>>> _internal_func()
NameError: "name '_internal_func' is not defined"
```

&emsp;而如果常规导入则不会受此影响：
```Python
>>> import my_module
>>> my_module.external_func()
23
>>> my_module._internal_func()
42
```

# 单末尾下划线 var_
&emsp;有时候名称被一个keyword占用了，命名的时候就不得不避开这些Python认定的关键字，下面的例子是用`class_`代替`class`来表示传递给方法的参数。
```Python
>>> def make_object(name, class):
SyntaxError: "invalid syntax"
 
>>> def make_object(name, class_):
...    pass
```

&emsp;其实就是为了避免冲突而采用的。

# 双前导下划线: __var
> A double underscore prefix causes the Python interpreter to rewrite the attribute name in order to avoid naming conflicts in subclasses.

&emsp;这将导致Python解释器重写属性名称避免在子类当中冲突。
&emsp;这其实也称作 *name mangling*. 解释器改变变量名称以使得后续扩展类的时候更不容易发生冲突。
例子：
```Python
class Test:
   def __init__(self):
       self.foo = 11
       self._bar = 23
       self.__baz = 23
```

使用内置的(built-in) [dir()](https://docs.python.org/3/library/functions.html#dir) 来查看：

```Python
>>> t = Test()
>>> dir(t)
['_Test__baz', '__class__', '__delattr__', '__dict__', '__dir__',
'__doc__', '__eq__', '__format__', '__ge__', '__getattribute__',
'__gt__', '__hash__', '__init__', '__le__', '__lt__', '__module__',
'__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__',
'__setattr__', '__sizeof__', '__str__', '__subclasshook__',
'__weakref__', '_bar', 'foo']
```
&emsp;查看列表，查找我们的原始变量`foo`,`_bar` 和 `__baz`我们注意到:
- self.foo变量在属性列表中显示为未修改为foo
- self._bar的行为方式相同 - 它以_bar的形式显示在类上。 就像我之前说过的，在这种情况下，前导下划线仅仅是一个约定。 给程序员一个提示而已。
- 然而对于`self.__baz`而言，情况看来不同。列表中并没有`__baz`这个名字的变量

&emsp；仔细观察，发现有个`_Test__baz`的属性。这就是Python解释器所做的名称修饰。 它这样做是为了防止变量在子类中被重写。

&emsp;而如果创建另一个扩展Test类的类，并尝试重写构造函数中添加的现有属性：

```python
class ExtendedTest(Test):
   def __init__(self):
       super().__init__()
       self.foo = 'overridden'
       self._bar = 'overridden'
       self.__baz = 'overridden'
```
foo, _bar, 和 __baz的值会出现在这个`ExtendedTest`类的实例上吗？ Let's take a look.

```python
>>> t2 = ExtendedTest()
>>> t2.foo
'overridden'
>>> t2._bar
'overridden'
>>> t2.__baz
AttributeError: "'ExtendedTest' object has no attribute '__baz'"
```
得到的`AttributeError`? 名称修饰被再次触发了！事实上，这个对象没有__baz属性：

&emsp;我们可以看到_baz变成_ExtendedTest__baz以防止意外修改：
```python
>>> t2._ExtendedTest__baz
'overridden'
```

&emsp;但原来的_Test__baz还在：
```python
>>> t2._Test__baz
42
```

&emsp；双下划线名称修饰对程序员完全透明。下面例子证实：
```python
class ManglingTest:
   def __init__(self):
       self.__mangled = 'hello'
 
   def get_mangled(self):
       return self.__mangled
 
>>> ManglingTest().get_mangled()
'hello'
>>> ManglingTest().__mangled
AttributeError: "'ManglingTest' object has no attribute '__mangled'"
```

还有另一个例子：
```python

_MangledGlobal__mangled = 23
 
class MangledGlobal:
   def test(self):
       return __mangled
 
>>> MangledGlobal().test()
```

&emsp;在这里声明了一个名为`_MangledGlobal__mangled`的全局变量。然后在名为MangledGlobal的类的上下文中访问变量。由于名称修饰，我们能够在类的test()方法内，以__mangled来引用_MangledGlobal__mangled全局变量。

&emsp;Python解释器自动将名称__mangled扩展为_MangledGlobal__mangled,因为它以两个下划线字符开头。这表明名称修饰不是专门与类属性关联的。它适用于在类上下文中使用的两个下划线字符开头的任何名称。

# 双前导和双末尾下划线 __var__
&emsp;如果一个名字同时以双下划线开始和结束，则不会应用名称修饰。 由双下划线前缀和后缀包围的变量不会被Python解释器修改：
```python
class PrefixPostfixTest:
   def __init__(self):
       self.__bam__ = 42
 
>>> PrefixPostfixTest().__bam__
42
```
&emsp;但是，Python保留了有双前导和双末尾下划线的名称，用于特殊用途。 这样的例子有，__init__对象构造函数，或__call__ --- 它使得一个对象可以被调用。

&emsp;但是，Python保留了有双前导和双末尾下划线的名称，用于特殊用途。 这样的例子有，__init__对象构造函数，或__call__ --- 它使得一个对象可以被调用。

# 单下划线 _
&emsp;有时候独立下划线是用作一个名字，来表示某个变量是临时的或无关紧要的。在下面循环中，我们不需要访问正在运行的索引，可以使用`_`来表示它只是一个临时变量
```python

>>> for _ in range(32):
...    print('Hello, World.')
```
&emsp;也可以在拆分(unpacking)表达式中将单个下划线用作“不关心的”变量，以忽略特定的值。 同样，这个含义只是“依照约定”，并不会在Python解释器中触发特殊的行为。 单个下划线仅仅是一个有效的变量名称，会有这个用途而已。

&emsp;在下面的代码示例中，我们将汽车元组拆分为单独的变量，但我只对颜色和里程值感兴趣。 但是，为了使拆分表达式成功运行，需要将包含在元组中的所有值分配给变量。 在这种情况下，`_`作为占位符变量可以派上用场：

```python
>>> car = ('red', 'auto', 12, 3812.4)
>>> color, _, _, mileage = car
 
>>> color
'red'
>>> mileage
3812.4
>>> _
12
```

&emsp;除了用作临时变量之外，“_”是大多数Python REPL中的一个特殊变量，它表示由解释器评估的最近一个表达式的结果。

&emsp;这样比较方便，可以在一个解释器会话中访问先前计算的结果，或者，你是在动态构建多个对象并与它们交互，无需事先给这些对象分配名字：
```python
>>> 20 + 3
23
>>> _
23
>>> print(_)
23
 
>>> list()
[]
>>> _.append(1)
>>> _.append(2)
>>> _.append(3)
>>> _
[1, 2, 3]
```

