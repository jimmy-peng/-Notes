# ORM简单框架
~~~
#!/usr/bin/env python
# -*- coding: utf-8 -*-

' Simple ORM using metaclass '

__author__ = 'Michael Liao'

class Field(object):
    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type
    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name)

class StringField(Field):
    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')

class IntegerField(Field):
    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')

class ModelMetaclass(type):
	 '''
	 cls: 正在实例化的类的对象
	 name: 具体类的类名
	 attrs: 具体类的属性字典，这里包括类的成员和方法
	 '''
    def __new__(cls, name, bases, attrs):
        # 创建model类时，也会调用这个元类,这里是过滤掉Model走这里
        if name=='Model':
            return type.__new__(cls, name, bases, attrs)
        print('Found model: %s' % name)
        mappings = dict()
        
        # 过滤出具体类中属于Field的属性字典，并保存在mappings中
        # k是属性名
        # v是具体类的值
        for k, v in attrs.iteritems():
            if isinstance(v, Field):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
        
        # 删除掉具体类中的所有属于Field的属性
        for k in mappings.iterkeys():
            attrs.pop(k)
        
        # 确定需要保存到具体类中的属性
        attrs['__mappings__'] = mappings # 保存属性和列的映射关系
        attrs['__table__'] = name # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)

class Model(dict):
	# 引用元类
    __metaclass__ = ModelMetaclass

    def __init__(self, **kw):
        super(Model, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.iteritems():
            fields.append(v.name)
            params.append('?')
            args.append(getattr(self, k, None))
        
        #生成sql语句
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))

# testing code:

class User(Model):
    id = IntegerField('uid')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')

u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
u.save()
~~~

### ORM原理

全名对象关系映射，本质上就是一个代码生成器。 

最基本的原理就是将声明的类通过类的不同具体声明映射成一个新的类  
这个新的类可以操作数据库的对象，屏蔽了Model层对数据库的操作细节

## 元类理解

Python会在类的定义中寻找__metaclass__属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类。把下面这段话反复读几次。

type也是一个元类，当你声明一个类的时候，type会在内存创建对应类的代码和数据

~~~
Foo中有__metaclass__这个属性吗？如果是，Python会在内存中通过__metaclass__创建一个名字为Foo       
类对象（我说的是类对象，请紧跟我的思路）。如果Python没有找到__metaclass__，它会继续在Bar（父
类）中寻找__metaclass__属性，并尝试做和前面同样的操作。如果Python在任何父类中都找不到
__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作。如果还是找不到.
__metaclass__,Python就会用内置的type来创建这个类对象。
~~~

你创建自己的元类的时候，最后还是需要将参数传入type函数，因为type函数担当了分配类内存的主要工作

一个函数也可以充当元类，不一定需要class关键字

~~~
# 元类会自动将你通常传给‘type’的参数作为自己的参数传入
def upper_attr(future_class_name, future_class_parents, future_class_attr):
    '''返回一个类对象，将属性都转为大写形式'''
    # 选择所有不以'__'开头的属性
    attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
    # 将它们转为大写形式
    uppercase_attr = dict((name.upper(), value) for name, value in attrs)

    # 通过'type'来做类对象的创建
    # uppercase_attr 这里面类属性大小写已经改变了
    return type(future_class_name, future_class_parents, uppercase_attr)

__metaclass__ = upper_attr  #  这会作用到这个模块中的所有类

class Foo(object):
    # 我们也可以只在这里定义__metaclass__，这样就只会作用于这个类中
    bar = 'bip'

# 测试
print hasattr(Foo, 'bar')
# 输出: False
print hasattr(Foo, 'BAR')
# 输出:True
 
f = Foo()
print f.BAR
# 输出:'bip'
~~~

~~~
# 请记住，'type'实际上是一个类，就像'str'和'int'一样
# 所以，你可以从type继承
class UpperAttrMetaClass(type):
    # __new__ 是在__init__之前被调用的特殊方法
    # __new__是用来创建对象并返回之的方法
    # 而__init__只是用来将传入的参数初始化给对象
    # 你很少用到__new__，除非你希望能够控制对象的创建
    # 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__
    # 如果你希望的话，你也可以在__init__中做些事情
    # 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用
    def __new__(upperattr_metaclass, future_class_name, future_class_parents, future_class_attr):
        # 选择所有不以'__'开头的属性
        attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
        # 通过'type'来做类对象的创建
        # uppercase_attr 这里面类属性大小写已经改变了

        uppercase_attr = dict((name.upper(), value) for name, value in attrs)
        return type(future_class_name, future_class_parents, uppercase_attr)
        
# 还有一种更OO的写法        
class UpperAttrMetaclass(type):
    def __new__(upperattr_metaclass, future_class_name, future_class_parents, future_class_attr):
        attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
        uppercase_attr = dict((name.upper(), value) for name, value in attrs)
 
        # 复用type.__new__方法
        # 这就是基本的OOP编程，没什么魔法
        return type.__new__(upperattr_metaclass, future_class_name, future_class_parents, uppercase_attr)

~~~

#### 这里为什么`upperattr_metaclas`是一个对象呢？
`__new__`是类的方法,方法的第一个参数永远是对象，哈哈

#### `__new__`和`__init__`有什么不一样？

`__new__`是用来分配内存的， `__init__`是用来初始化的

## 感悟
#### 重新思考什么是对象，什么是类

如果把对象看做是数据和代码的混合体，
类其实也是代码和数据的混合体

#### 为什么类和对象的界限可以模糊？  
类和对象实质上是没有区别的，我们一般说对象是类的映射体，也就是由一段代码产生的另一段代码而已  
但是别忘了还有元类，元类映射出来的是类，类是元类的对象，所以类和对象的区别其实没那么明显

由此可以推广到所有语言的类型系统  
int类型和string类型都支持系统的操作函数 `+`  
从C语言的角度看，类型就是语义化的操作，对不同的类型的操作，语义是不一样的  
类型的大小也是语义化操作的一部分，系统基本类型大小是定死的  
可以把操作符看做函数  
自定义的类型大小是有基本类型组合而成的(实际上还和编译器有关)


