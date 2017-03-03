# 继承、接口和泛型

## 前言

面向对象三大基本特性：封装、继承、多态。上一篇中介绍了类的定义，下面就了解下F#中继承和多态的使用吧。😋

同样的，面向对象的基础概念不多说，就对比下语法，并简单说明下应该注意的地方。

## 继承

### 对象表达式（Object Expressions）

在介绍继承之前，先介绍一下F#面向对象中常用的一个表达式：**对象表达式**。它用于**基于现有类型**创建匿名对象类型，比如有时候只希望在一个或少数几个对象中修改成员函数的实现，这时候不一定要重新定义一个现有类型的派生类然后实例化。

可在创建对象时通过关键字`with`提供新的函数实现代码：

```F#
let oriPt = {
    new Point2D() with 
        member __.ToString() = "我是原点"
}
```

`oriPt`使用对象表达式实例化，并重写了基类的`ToString`方法。而如果在C#中我们就需要先定义一个派生类，然后重写``ToString``方法。其实这也是继承，只是这样会减少创建新的命名类型所需的代码和开销。

对象表达式需要放在一对大括号（`{}`）中，其中的**`new`不可省略**，且**修改的成员函数必须是虚方法（包括抽象方法）**。

### 继承的实现

继承是面向对象的一大特性，下面分别是C#和F#中的语法对比。定义一个继承于`Point2D`的类：

```C#
public class Particle : Point2D
{
    public double Mass { get; set; }
}
```

```F#
type NamedPoint2D() =
    inherit Point2D()
    member val Name = "pt2d" with get, set
```

在F#中，在F#中，派生类中使用关键字`inherit`指定所继承的基类及其构造函数，若子类需要调用基类方法，同样使用**`base`**关键字。*`base`无法像`this`一样自定义。*😃

如果有多个构造函数，通常可以在**不使用主构造函数**的情况下使用**对象表达式**返回对象，或者使用使用其他构造函数实例化返回，并用`then`关键字指定额外的执行语句。

```
type NamedPoint2D =
    inherit Point2D
    val mutable Name:string
    new (x,y) ={
        inherit Point2D(x,y)
        Name = ""
    }
    new (name) = {
        inherit Point2D();
        Name = name
    }
    new (x,y,name) as this = 
        NamedPoint2D(x,y)
        then
            this.Name <- name
```


## 接口

### 接口的定义和使用

抽象类决定一个对象**“是什么”**，而接口决定了一个对象**“具有什么功能”**。我们先看F#中的定义：

```F#
type I2DLocation = interface		//完整定义方式
    abstract member X : float with get, set
    abstract member Y : float with get, set
end
type I2DLocation =					//简要定义方式
    abstract member X : float with get, set
    abstract member Y : float with get, set
```

在F#中，若不使用`interface`关键字显示定义，**只要类的所有成员都为抽象（`abstract`）的，就会被类型推断系统推断为接口**。

**接口在F#需要显示实现**，所以在实现接口后，使用时也需要**转换至接口类型**，否则无法调用接口的属性或方法。

```F#
type Point2D(xValue:double, yValue:double) as this=
	……	//省略其他代码
	interface I2DLocation with
        member this.X with get() = this.X and set(v) = this.X <- v
        member this.Y with get() = this.Y and set(v) = this.Y <- v
let pt = Point2D()
let l = pt :> I2DLocation	//使用向上转换符，因为实现的接口已在编译期确定
printfn "x=%f,y=%f" l.X l.Y
```

### 使用对象表达式实现接口的匿名类型

[对象表达式](#对象表达式（Object Expressions）)不仅可基于类创建匿名类型，同样可基于接口创建匿名类型。在C#中，我们在基于`IComparer`比较器进行比较时，都要定义一个实现`IComparer`的类。而在F#中，因为有对象表达式，我们可以省去很多代码。假设我们将一些`Point2d`基于X坐标排序：

```F#
open System.Collections.Generic
let pts = List<_>(
    [| Point2D();Point2D(4.,2.); Point2D(3.,4.);Point2D(6.,2.); |]
)
pts.Sort({new IComparer<Point2D> with 
    member __.Compare(l, r) =
        int(l.X - r.X)
})
```

代码使用`List<T>`定义了几个点，然后使用对象表达式定义了一个匿名类的实例传入Sort方法。而这样不需要像C#中重新定义一个类型。*虽然这样的功能在C#中经常使用的是Linq中的Sort，但我们现在介绍面向对象就先以这为例了。*

可以看到基于接口定义的对象表达式跟基于类的有所不同，在**接口名后不能使用参数**，因为接口无法实例化。

### IDisposable接口

说到接口，就该说下.NET中比较特殊的`IDisposable`，实现了此接口的对象必须实现`Dispose`函数成员，用于对对象执行显式的销毁操作，通常执行一些释放资源的代码。

在C#中，实现`IDisposable`的对象可用`using`进行自动销毁。在F#中，则有对应的`use`关键字和`using`函数，而且在实例化实现了`IDisposable`接口的对象，必须使用`new`关键字。

```F#
open System; open System.Data.SqlClient
type Database(conStr) =
    let con = new SqlConnection(conStr)	//new不可省略
    member __.ConnectionString = conStr
    member __.Connect() = con.Open()
    member __.Close() = con.Close()
    interface IDisposable with
        member this.Dispose() = this.Close()

//使用use关键字，与C#类似
let testIDisposable() =
    use db = new Database("connection string ...")
    db.Connect()

//使用using函数，第一个参数是IDisposable接口的对象，第二个是要执行的操作
let testUsing(db:Database) = db.Connect()
using (new Database("connection string ...")) testUsing
```

-   使用`use`时，会在`use`所在的**代码块结束**时调用`Dispose`方法，在示例中，是在`testIDisposable`执行完毕时。
-   `using`函数会在它的**函数参数执行完毕**时调用`Dispose`方法，示例中是在`testUsing`函数执行完毕时。

通常，应该选择使用`use`。但要注意的是，因为`use`需要等代码块结束时进行操作，所以**无法在模块中使用**，若在模块中使用，只会被当成**`let`**，而不会自动销毁对象。在模块中使用，可以用`using`函数。


## 类型转换与扩展

### 类型转换

在[数值运算和流程控制语法](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-2.html)中我们介绍F#中数值转换需要使用对应的函数，如转成`Int`类型使用`int`函数等。

但基类和子类之间的转换，F#提供`upcast`（子类转为基类）和`downcast`（基类转为子类）函数进行转换，或者使用对应的符号函数：`:>`和`:?>`。

```F#
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

`:?`还可以用在模式匹配（[数值运算和流程控制语法](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-2.html)有介绍过，类似于C#中的`switch`）里。

```F#
match shape with
| :? Circle    as c -> printfn "circle with radius %f" c.Radius
| :? Rectangle as r when r.Length = r.r.Height 
                    -> printfn "%f x %f square"    r.Length r.r.Height
| :? Rectangle as r -> printfn "%f x %f rectangle" r.Length r.r.Height
| _                 -> printfn "<unknown shape>"
| null              -> raise (ArgumentNullException("shape"))
```

此段代码从中 ["What’s New in C# 7.0"](https://blogs.msdn.microsoft.com/dotnet/2016/08/24/whats-new-in-csharp-7-0/) （中文翻译[《[C＃7.0中有哪些新特性？](http://www.cnblogs.com/powertoolsteam/p/whats-new-in-C-sharp-7-0.html)》]）C#7.0的模式匹配的示例代码转换而来的。C#原代码如下：

```C#
switch(shape)
{
    case Circle c:
        WriteLine($"circle with radius {c.Radius}");
        break;
    case Rectangle s when (s.Length == s.Height):
        WriteLine($"{s.Length} x {s.Height} square");
        break;
    case Rectangle r:
        WriteLine($"{r.Length} x {r.Height} rectangle");
        break;
    default:
        WriteLine("<unknown shape>");
        break;
    case null:
        throw new ArgumentNullException(nameof(shape));
}
```

C#的代码与F#一样，最后一项`null`其实是无法被匹配到的。F#中以**“`_`”**作为**通配符**。

可以发现C#最近几个大版本中的函数式新功能是借鉴于F#的，C#在函数式的道路是越走越远了。👍

*F#中在当前4.0版本中还没有`nameof`操作符，已经实现，估计会在新版本中释出。而C#7.0的功能也可以在[Visual Studio “15” Preview 4](https://www.visualstudio.com/news/releasenotes/vs15-relnotes)中体验。*

### 装箱（Boxing）和拆箱（Unboxing）

F#中使用`box`和`unbox`函数进行装箱和拆箱操作：

```F#
let i1 = 4
let o = box i	//此时o为obj类型
let i2 ： int = unbox o
```

在F#中，`obj`为`System.Object`的别名。

### 扩展

可以使用`with`关键字对现有**类型和接口**增加属性及方法。

```F#
type System.Int32 with
    member i.IsPrime with get () =
        Array.forall (fun x-> i%x <> 0) [| 2..i/2 |]
(250).IsPrime        //250不是质数，将为false
```

示例中给`int`类型添加一个属性用于判断其是否为质数。 

##结构和枚举

### 结构（Struct）

在前面介绍的面向对象类以及类涉及的相关内容，但在示例的代码感觉使用类并没有感觉有什么优势。其实像上一篇中的`Point2D`类，使用结构（`Struct`）也许会更好一些。

结构是值类型，与类的引用类型不同，在内存分配上是被分配在**栈（Stack）**上，所以在使用中内存消耗更少，而且不需要垃圾回收（GC）。这方面知识熟悉.NET框架的大家都很熟悉了。

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

结构定义与类一样，在内容不为空时可省略`struct end`关键字。不过这样就和类定义一样了，所以**需要加上`[<Struct>]`特性**以示区别。这和抽象类、密封类的定义方法一致。

### 枚举（Enum）

枚举在F#中比较少用，替代的是使用**可区分联合**（Discriminated Unions，常被称作DU）。枚举可看作是可区分联合的简化，它们之间的区别等介绍可区分联合时再说明。下面是枚举的定义及与其基础类型的转换：

```F#
type Card =
    | Jack = 11
    | Queen = 12
    | King = 13
    | Ace = 14
let q = enum<Card>(12)	//int转为enum类型
let i = int Card.King	//enum转为int类型
```

## 泛型及约束

F#与C#同样基于.NET，所以泛型也并没有什么特殊的。在使用上，F#中的**类型参数需要以“`'`”（单引号）开头**。

可以在类、结构、接口、集合、函数等中使用泛型。

```F#
let print<'a> (x:'a) = 
    printfn "%A" x
type MyClass<'T> (y:'T) =    
    member val Y = y with get, set
```

泛型在定义和使用上都与C#类似，F#中类型参数一般使用`'a` 、`'b`……

但泛型约束就与C#有较大的区别了，以下是C#与F#泛型约束的对比表。

| C# 约束              | F# 约束                              | 描述                                       |
| ------------------ | ---------------------------------- | ---------------------------------------- |
| `where T: struct ` | `when 'T : struct `                | 值类型。                                     |
| `where T : class ` | `when 'T : not struct `            | 引用类型。                                    |
| `where T : new() ` | `when 'T : ( new : unit -> 'a ) `  | 构造函数约束。C#中此约束必须放在最后，但F#中**不**需要。         |
| `where T : <基类>`   | `when 'T :> type `                 | T类型参数必须是从指定的基类型派生的，基类型可以是接口。             |
| `where T : U`      | **不支持**                            | T必须继承自U。                                 |
| **不支持**            | `when 'T : null `                  | 提供的类型必须可以为`null`， 这包括所有 .NET 对象类型。       |
| **不支持**            | `when 'T or 'U : (member 成员签名) `   | **显式成员**约束，所提供的类型参数T和U中至少有一个必须包含指定签名的成员。 |
| **不支持**            | `when 'T : enum<基础类型> `            | 枚举类型约束，提供的类型必须是基于指定基础类型的枚举。              |
| **不支持**            | `when 'T : delegate<tuple参数,返回类型>` | 提供的类型必须是具有指定的参数和返回值的委托类型。其中参数是一个Tuple。   |
| **不支持**            | `when 'T : comparison `            | 提供的类型必须支持比较。                             |
| **不支持**            | `when 'T : equality `              | 提供的类型必须支持相等性。                            |
| **不支持**            | `when 'T : unmanaged `             | 提供的类型必须是**非托管类型**。 非托管类型是某些基元类型（`sbyte`、`byte`、`char`、`nativeint`、`unativeint`、`float32`、`float`、`int16`、`uint16`、`int32`、`uint32`、`int64`、`uint64 或 decimal`）、枚举类型、`nativeptr<_>` 或其**所有字段均为非托管类型**的非泛型结构。 |

在F#，泛型约束使用`when`关键字，写在`<>`里面或者外面均是可以的。*虽然F#支持着很多C#不支持的约束，但其实这些约束很少用到。*😏

其中显式成员约束可用于实现鸭子类型，有兴趣可通过文章《[方法多态与Duck typing；C#之拙劣与F#之优雅](http://blog.csdn.net/hikaliv/article/details/4559927)》了解。



---

*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 转载请注明源链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-7.html>。*

