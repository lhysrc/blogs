

# 在F#中使用面向对象编程

[TOC]

面向对象的思想已经非常成熟，而使用C#的程序员对面向对象也是非常熟悉，所以我就不对面向对象进行介绍了，在这篇文章中将只会介绍面向对象在F#中的使用。

F#是支持面向对象的函数式编程语言，所以你用C#能做的，用F#也可以做，而且通常代码还会更为**简洁**。我们先看下面这个用C#定义的类，然后用F#来定义。

```C#
//定义一个二维点
[DebuggerDisplay("({X}, {Y})")]
public class Point2D
{
    // 数量
    private static int count = 0;
    // 原点
    public readonly Point2D OriginalPoint = new Point2D(0, 0);
    // 完整属性X
    private double x;
    public double X
    {
        get { return this.x; }
        set { this.x = value; }
    }
    // 自动属性Y
    public double Y { get; set; }
    // 到原点的距离
    public double LengthToOriginal
    {
    	get { return this.GetDistance(this.OriginalPoint); }
    }
    // 默认构造函数
    public Point2D() : this(0,0) {}
    // 包含两个参数的构造函数
    public Point2D(double x, double y)
    {
        this.X = x;
        this.Y = y;
        count++; // 创建新点时，数量递增1
    }
    // 将(x,y)数值转为Point2D对象的静态方法
    public static Point2D FromXY(double x, double y)
    {
		return new Point2D(x, y);
    }
    // 计算到指定点的距离
    public virtual double GetDistance(Point2D point)
    {
        var xDif = this.X - point.X;
        var yDif = this.Y - point.Y;
        var distance = Math.Sqrt(xDif * xDif + yDif * yDif);
        return distance;
    }
    //重写ToString方法
    public override string ToString()
    {
    	return String.Format("Point2D({0}, {1})", this.X, this.Y);
    }
}
```

## 类

在F#中定义类不需要`class`关键字，除非定义一个**空**的类。

```F#
type Point2D() = class			//定义一个空类
	end
type internal Point2D() = class	//定义一个空的internal类
	end
type Point2D() =				//定义一个只包含字段x的类
	let mutable x = 0.0
```

不像C#，在F#中，类的访问修饰符默认是`public`的，所以`internal`就需要手动指定。并且，在F#，**不支持`protected`访问修饰符**，在F#中应该使用接口，对象表达式和高阶函数来实现类似功能。

### 字段（Field）

在上一例代码中我们已经定义了一个字段`x`，但类中用`let`定义的字段（包括后面介绍的函数）均为`private`的。若需要定义其他访问修饰符的字段，需要使用`val`定义。

```F#
type Point2D() =
    [<DefaultValue>] val mutable public x : float
```

其中`DefaultValue`特性用于将支持零值初始化的类型（具有零值的基元类型、可为null的类等）字段初始化为零值。

### 属性（Property）和索引器

使用`member`关键字定义属性，注意F#中的属性需要有一个**对象标识符**（类似C#中的`this`）作为前缀。

```F#
type Point2D() =
    let mutable x = 0.0
    // 定义属性X，其中set为private的。
    member this.X with get()  = x
                  and private set(v) = x <- v
    member val Y = 0.0 with get, set	//自动属性
```

在F#中，`this`也可以使用其他名称来代替。除了`this`（C++和C#的习惯），有的人还使用`self`（Python等的习惯）、`me`（VB.NET的习惯）等。若不需要在属性或方法内部使用到，一般还习惯使用`__`（两个下划线）来作为对象标识符。

而像C#`set`方法中的`value`，F#中也需要自己指定，如例子中使用`v`。当然`get`和`set`都可以省略使之成为只写或只读属性。

遗憾的是自动属性中不支持分别定义访问修饰符（如C#中的`{get; private set}`），只能使用完整属性来定义。

### 方法（Method）

方法同样使用`member`定义，而静态方法只需在`member`前添加`static`关键字。

```F#
type Point2D() =
    ……  //因为篇幅省略属性X,Y的代码
    // 定义一个函数来获取指定点到当前点的距离
    member this.GetDistance (point:Point2D) =
        let xDif = this.X - point.X
        let yDif = this.Y - point.Y
        let distance = sqrt (xDif**2. + yDif**2.)
        distance
    static member FromXY (x:double, y:double) =
        let point = Point2D()
        point.X <- x
        point.Y <- y
        point
```

### 构造函数与实例化

F#中有两种构造函数，一为**主构造函数**，上面例子中在**类型定义后面**的`()`即表示一个无参构造函数。

另一构造函数是可选的，在类里定义一个`new`函数即可。但`new`函数必须满足以下条件：

-   返回类型必须为该类
-   不使用`member`和对象标识符。
-   使用**一个**元组或`unit`作为参数。

我们改造上面的代码：

```F#
type Point2D (xValue:double, yValue:double) =
    let mutable x = xValue
    member val Y = yValue with get, set
    new() = Point2D(0.0,0.0)
```

其中，调用无参构造函数时，则使用默认构造函数实例化，且两个参数均为`0.0`。

若要为构造函数添加访问修饰符，写在其之前即可。

```F#
type internal Point2D internal (xValue:double, yValue:double) =
    private new() = Point2D(0.0,0.0)
```

**需要注意的是，当未定义构造函数时，F#并不像C#那样有一个无参的构造函数。**

在类中的`let`绑定会在调用**主构造函数**时运行，但如果需要在主构造函数中执行其他操作，需要使用`do`绑定。

```F#
type Point2D(xValue:double, yValue:double) =
    let mutable x = xValue
    static let mutable count = 0
    do
        count <- count + 1
```

注意`let`代码和`do`代码必须在`member`之前。而`do`语句中必须返回`unit`，可使用`ignore`函数丢弃返回值。

非静态的`do`语句会在实例化时执行，而静态的`do`语句会在第一次使用该类型时执行。

如果需要在`do`语句中访问其他方法，则需要**类级别的对象标识符**，在类型定义后使用`as`关键字指定；若想在非主构造函数中执行额外的代码，使用`then`关键字：

```F#
type Point2D (xValue:double, yValue:double) as self =
    do	//省略其他代码
        self.Print()
    new() = 
        Point2D(0.0,0.0)
        then printfn "创建原点。"
    member this.Print() = printfn "X=%f,Y=%f" this.X this.Y
```

但这样有一个问题：在执行`do`代码访问`Print`函数时，需要`self`已经实例化好，但因为`Y`自动属性，编译时会在`do`后面插入一个`Y@`字段（即Y的back-end字段），此时并未初始化。

所以，若在构造函数中需要访问类中的方法，只能将`Y`也更改为完整属性，并且`x`和`y`字段的`let`绑定必须在`do`之前。

#### 实例化

在上面代码中的`new()`构造函数中，实例化了一个`Point2D(0.0,0.0)`对象。在F#中，**实例化**时可以不使用`new`关键字，但有特例，在后面会介绍。

在C#中，我们可以使用**对象初始化器（Object Initializers）**在初始化时对属性进行赋值，在F#中，也有类似功能：

```C#
var pt = new Point2D() {X = 3.0};	//C#中使用对象初始化器
```

```F#
let pt = Point2D(X=3.0)				//F#中使用对象初始化器
```

到此，可将上面的`FromXY`方法进行简化：

```
static member FromXY (x:double, y:double) =
        Point2D(x,y)
```

### 索引器（Indexer）及切片（Slice）

#### 索引器

索引器，顾名思义就是可以使用索引来操作对象。在F#中，只需定义一个名为`Item`的属性或方法即可让该类的对象使用索引器。当然，若Item为方法，则索引器为只读的。

我们用下来的代码定义一个可变的字符串类。

```F#
open System.Collections.Generic
type WordBuilder(startingLetters : string) =
    let m_letters = new List<char>(startingLetters)
    member this.Item
        with get idx   = m_letters.[idx]
        and  set idx c = m_letters.[idx] <- c
    member this.Word = new string (m_letters.ToArray())

let wb = WordBuilder("I Love C#")
wb.[7] <- 'F'
printfn "%s" wb.Word	//输出为："I Love F#"
```

注意在使用索引访问时，需要使用**点**访问（`.[]`）。

#### 切片

切片和索引器类似，不过索引器访问的是一个元素，而切片访问的是数据的集合。实现切片访问需要添加`GetSlice`方法，切片是只读的。

```F#
open System.Collections.Generic
type WordBuilder(startingLetters : string) =
	…… //其他代码省略
    member this.GetSlice(lower:int option,upper:int option) =
        letters 
        |> Seq.skip (lower.Value)
        |> Seq.take (upper.Value - lower.Value + 1)
        |> Seq.toArray    	

let wb = WordBuilder("I Love C#")
printfn "%A" wb.[2..5]		//输出为：[|'L'; 'o'; 'v'; 'e'|]
```

此切片方法并未实现不完整，但已可了解实现方法。注意`GetSlice`的参数必须是`'a option`类型， 其中`option`类型将在后面再介绍，可理解为类似于`Nullable<int>`。

## 继承和扩展

### 继承

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













### 抽象类（Abstract Class）和密封（Sealed）

要将类定义为抽象类或密封类，只需要在类定义前加上`[<AbstractClass>]`或`[<Sealed>]`特性。

### 抽象方法和虚方法

在F#中，没有提供`virtual`关键字来定义虚方法。而使用定义一个抽象方法，并提供默认实现来代替。

```F#
type Point2D(xValue:double, yValue:double) as this=
    ……
    // 实现开头C#代码中的GetDistance虚方法，此处省略其他代码
    abstract GetDistance : point:Point2D -> double
    default this.GetDistance(point) =
        let xDif = this.X - point.X
        let yDif = this.Y - point.Y
        let distance = sqrt (xDif**2. + yDif**2.)
        distance
```

关键字`abstract`用来定义抽象方法，只给出**函数签名**，并且函数名前**不需使用对象标识符**；而`default`默认实现语句中也不需使用`member`关键字。

与C#一样，在派生类中进行override关键字重写基类中的虚方法：

```
type Point2D(xValue:double, yValue:double) =
    ……
    //重写Object类中的ToString虚方法，此处省略其他代码
    override this.ToString() = sprintf "Point2D(%f, %f)" this.X this.Y
```

在C#中，若要重写非虚方法，可使用`new`关键字。F#中没有此关键字，但仍然可以重写非虚方法，只是编译器会给出警告。

### 类型转换

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

#### 装箱（Boxing）和拆箱（Unboxing）

F#中使用`box`和`unbox`函数进行装箱和拆箱操作：

```
let i1 = 4
let o = box i	//此时o为obj类型
let i2 ： int = unbox o
```

在F#中，`obj`为`System.Object`的别名。

### 泛型及约束



### 扩展

#### 类级扩展

#### 对象级扩展




## 接口



##结构和枚举















---

内容较多，但熟悉面向对象的朋友其实只需了解F#与C#间语法的差别即可。



*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 转载请注明源链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-6.html>。*










