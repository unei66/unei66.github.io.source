---
layout: post
title:  "ruby基础语法"
date:   2018-04-17
categories: ruby
---

## 变量
+ 局部变量
    以英文字母或下划线开头
+ 全局变量
    以$开头
+ 实例变量
    以@开头
+ 类变量
    以@@开头

## 方法
+ 真假
    真：false，nil以外所有的对象
    假：false，nil
+ 约定：返回真假值的方法都要以 ? 结尾

## case 和＝＝＝
在case语句中,when判断值是否相等时,实际是使用===运算符来判断的。左边是数值或者字符串时,===与case的意义是一样的,除此以外, 还可以与=~ 一样用来判断正则表达式是否匹配,或者判断右边的对象是否属于左边的类,等等。对比单纯的判断两边的值是否相等,=== 能表达更加 广义的“相等”。

## if/unless
+ if/unless 可以写在希望执行的代码的后面。
    puts "a>b" if a>b

## 对象的同一性
+ 所有的对象都有标识和值。
+ 标识（ID）用来表示对象同一性。Ruby中所有的对象都是唯一的，对象的ID可以通过object_id或__id__方法取得。
+ 可以用equal? 判断两个对象是否是同一个对象。
+ 只要对象的字符串内容相等，就认为对象的值相等，可以用==或eql?判断
+ 


## yield
调用方法时传进来的块会在yield处被执行。

## instance_of?/is_a?
+ instance_of? 判断实例是否属于某个类
+ is_a? 判断实例是否属于某个类，或者是否是某类的子类


## 对象
```
class World
    # 类常量
    Age=100
    # 类变量
    @@gogo="fuck"
    # 初始化方法
    def initialize(name="hello")
        @name=name
    end

    # 实例方法
    def hello
        p "hello ,I'm #{@name}"
    end

    # getter
    def name
        @name
    end

    # setter
    def name=(value)
        @name=value
    end

    attr_reader :age
    attr_writer :age

    # 类方法
    class <<self
        def clazzMethod2(name)
            p "#{name} said hello too"
        end
    end

    # 类方法
    def World.clazzMethod3(name)
        p "#{name} said hello third"
    end

    def self.method4(name)
        p "#{name} i am method4"
    end

    # private
    def methodPrivate(name)
        p "#{name} i am methodPrivate"
    end

    private :methodPrivate
end

# 外部定义类方法
class <<World
    def clazzMethod(name)
        p "#{name} said hello"
    end
end

gogo=World.new("haha")
gogo.hello

gogo.name="heihei"
gogo.hello
gogo.age =19
p gogo.age


World.clazzMethod("hello")
World.clazzMethod2("world")
World.clazzMethod3("gogo")
p World::Age
World.method4("heihei")
# gogo.methodPrivate("private")

# 单例方法
class << gogo
    def fuck
        p "fuck ,#{self.name}"
    end
end

world2=World.new("world2")

gogo.fuck
world2.fuck

```

## module mix-in
```
module HelloModule
    Version = "1.0"

    def hello(name)
        p "Hello ,#{name}."
    end

    module_function :hello
end

p HelloModule::Version
HelloModule.hello("Alice")

class Hello
    include HelloModule
end

h=Hello.new
h.hello("module")
```

+ include? 类是否包含某个模块
+ ancestors 类的继承关系
+ superclass 类的继承关系

## 方法是用原则
+ 同继承关系一样,原类中已经定义了同名的方法时,优先使用该方法
+ 在同一个类中包含多个模块时,优先使用最后一个包含的模块。
+ 嵌套include时,查找顺序也是线性的
+ 相同的模块被包含两次以上时,第 2 次以后的会被省略

## 异常
+ begin - rescue=>ex - ensure - end
```
begin
    xxxx
rescue=>ex
    p ex.message
    retry #重试begin
ensure
    xxx.close
end
```

+ statement1 rescue statement2

## 反射
1. instance_eval class_eval
    与eval区别，这些方法会在指定对象或模块的上下文中对代码进行求值，该对象或模块成为代码求值时的self的值。第二个区别是它们可以对代码块求值。
+ instance_eval 为对象定义一个单键方法，如果对象为类，该方法则为类方法
+ class_eval 定义一个普通的实例方法

## private
+ 如果调用方法的接收者不是自己，则必须明确指明一个接收者
+ 私有方法只能被隐含接收者调用。
