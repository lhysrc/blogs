# 面向对象编程之继承、接口与值类型

## 前言

面向对象三大基本特性：封装、继承、多态。继承

## 继承

继承是面向对象的一大特性，下面分别是C#和F#中的语法对比。定义一个继承于`Point2D`的类：

```C#
public class Particle : Point2D
{
    public double Mass { get; set; }
}
```

```F#
type Particle() =
    inherit Point2D()
    member val Mass = 0. with get, set
```

在C#中，构造函数若要调用基类方法









## 类型转换

在[数值运算和流程控制语法](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-2.html)中我们介绍F#中数值转换需要使用对应的函数，如转成`Int`类型使用`int`函数等。

但基类和子类之间的转换，F#提供`upcast`（子类转为基类）和`downcast`（基类转为子类）函数进行转换，或者使用对应的符号函数：`:>`和`:?>`。

```
type Base() = class end							//定义基类
type Derived() = inherit Base()					//定义子类
let myDerived = Derived()						
let upCaseResult = myDerived :> Base			//使用:>转换为基类
let upCaseResult2 : Base = upcast myDerived		//使用upcast转换为基类
let downCastResult = upCastResult :?> Derived	//使用:?>转换为子类
let downCastResult2 : Derived = downcast cast	//使用downcast转换为子类
```

需要注意的是，upcast操作总是安全的；但downcast并一定成功，可使用`:?`在转换前进行类型判断，否则转换失败会引发`InvalidCastException`异常。

```F#
if upCastResult :? Derived then upCastResult :?> Derived
```

### 装箱（Boxing）和拆箱（Unboxing）

F#中使用`box`和`unbox`函数进行装箱和拆箱操作：

```
let i1 = 4
let o = box i	//此时o为obj类型
let i2 ： int = unbox o
```

在F#中，`obj`为`System.Object`的别名。

## 

## 扩展

### 类级扩展

### 对象级扩展




## 接口



##结构（Struct）

在前面介绍的面向对象类以及类涉及的相关内容，但在示例的代码感觉使用类并没有感觉有什么优势。其实像上一篇中的`Point2D`类，使用结构（`Struct`）也许会更好一些。

结构是值类型，与类的引用类型不同，在内存分配上是被分配在栈（Stack）上，所以在使用中内存消耗更少，而且不需要垃圾回收（GC）。这方面知识熟悉.NET框架的大家都很熟悉了。

下面是结构的定义：

```F#
type Point2D(xValue:double, yValue:double) = struct
    member this.X = xValue
    member this.Y = yValue
end
[<Struct>]
type Point2D(xValue:double, yValue:double) =
    member this.X = xValue
    member this.Y = yValue
```

结构定义与类一样，在内容不为空时可省略`struct end`关键字。不过这样就和类一样了，所以**需要加上`[<Struct>]`特性**以示区别。这和抽象类、密封类的定义一致。





## 枚举（Enum）





## 泛型及约束


