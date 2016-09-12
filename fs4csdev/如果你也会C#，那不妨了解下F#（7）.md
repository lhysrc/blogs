# 在F#中使用OOP：继承、接口与泛型


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

## 泛型及约束



## 扩展

### 类级扩展

### 对象级扩展




## 接口



##结构和枚举















---


