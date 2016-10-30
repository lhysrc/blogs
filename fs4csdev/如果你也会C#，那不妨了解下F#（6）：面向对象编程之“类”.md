# 在F#中使用OOP之“类”

[TOC]
##前言

面向对象的思想已经非常成熟，而使用C#的程序员对面向对象也是非常熟悉，所以我就不对面向对象进行介绍了，在这篇文章中将只会介绍面向对象在F#中的使用。

F#是支持面向对象的函数式编程语言，所以你用C#能做的，用F#也可以做，而且通常代码还会更为**简洁**。我们先看下面这个用C#定义的类，然后用F#来定义。

```C#
//定义一个二维点
[DebuggerDisplay("({X}, {Y})")]
public class Point2D
{
    // 用于统计已实例化的数量
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

## 定义类

在F#中定义类不需要`class`关键字，除非定义一个**空**的类。

```F#
type Point2D() = class			//定义一个空类
	end
type internal Point2D() = class	//定义一个空的internal类
	end
type Point2D() =				//定义一个只包含字段x的类
	let mutable x = 0.0
```

不像C#，在F#中，类的访问修饰符**默认是`public`**的，所以`internal`就需要手动指定。并且，在F#，**不支持`protected`访问修饰符**，在F#中应该使用接口，对象表达式和高阶函数来实现类似功能。

## 字段（Field）

在上一例代码中我们已经定义了一个字段`x`，但类中用`let`定义的字段（包括后面介绍的函数）均为`private`的。若需要定义其他访问修饰符的字段，需要使用`val`定义。

```F#
type Point2D() =
    [<DefaultValue>] val mutable public x : float
```

其中`DefaultValue`特性用于将支持零值初始化的类型（具有零值的基元类型、可为null的类等）字段初始化为零值。

## 属性（Property）和索引器

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

## 方法（Method）

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

### 重载方法

F#类中的方法重载与C#一样，只需重新定义一个同名成员函数，且签名不同即可。下面实现一个接受`int`类型的`FromXY`方法：

```F#f
static member FromXY (x:int, y:int) =
        Point2D.FromXY(float x,float y)     
```

## 构造函数与实例化

F#中有两种构造函数，一为**主构造函数**（也称隐式构造函数），上面例子中在**类型定义后面**的`()`即表示一个无参构造函数。

另一种构造函数（显示定义）是可选的，在类里定义一个`new`函数即可。但`new`函数必须满足以下条件：

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

其中，调用无参构造函数时，则使用主构造函数实例化，且两个参数均为`0.0`。

若要为构造函数添加访问修饰符，写在其之前即可。

```F#
type internal Point2D internal (xValue:double, yValue:double) =
    private new() = Point2D(0.0,0.0)
```

**需要注意的是，在类型定义之后若不提供主构造函数，F#并不像C#那样有一个无参的构造函数。**

下面的代码能通过编译，但你无法进行实例化：

```F#
type Point2D = class end
```

在类中的`let`绑定会在调用**主构造函数**时运行，但如果需要在主构造函数中执行其他操作，需要使用`do`绑定。

```F#
type Point2D(xValue:double, yValue:double) =
    let mutable x = xValue
    static let mutable count = 0
    do
        count <- count + 1
```

注意`let`代码和`do`代码必须在`member`之前。而`do`语句中必须返回`unit`，可使用`ignore`函数丢弃返回值。

非静态的`do`语句会在实例化时执行，而静态的`do`语句会在第一次使用该类型时执行。而在**没有主构造函数的类中，无法使用`let`和`do`语句**。

如果需要在`do`语句中访问其他方法，则需要**类级别的对象标识符**，在类型定义后使用`as`关键字指定；若想在非主构造函数中执行额外的代码，使用`then`关键字：

```F#
type Point2D (xValue:double, yValue:double) as self =
    do	//省略其他代码
        self.Print "在主构造函数中。"
    new() as this= //主构造函数中使用self，此处用this。定义为任何名称都可以
        Point2D(0.0,0.0)
        then this.Print "在无参构造函数中。"
    member this.Print str = printfn "%s" str
```

但这样有一个问题：在执行`do`代码访问`Print`函数时，需要`self`已经实例化好，但因为`Y`自动属性，编译时会在`do`后面插入一个`Y@`字段（即Y的back-end字段），此时并未初始化。即违反了**“先定义后引用”**的原则。

所以，若在构造函数中需要访问类中的方法，只能**将`Y`也更改为完整属性**，并且`x`和`y`字段的`let`绑定必须在`do`之前。

### 实例化

在上面代码中的`new()`构造函数中，实例化了一个`Point2D(0.0,0.0)`对象。在F#中，**实例化**时可以不使用`new`关键字，但有特例，在后面会介绍。

在C#中，我们可以使用**对象初始化器（Object Initializers）**在初始化时对属性进行赋值，在F#中，也有类似功能：

```C#
var pt = new Point2D() {X = 3.0};	//C#中使用对象初始化器
```

```F#
let pt = Point2D(X=3.0)			//F#中使用对象初始化器，注意此处调用的是无参构造函数
```

到此，可将上面的`FromXY`方法进行简化：

```
static member FromXY (x:double, y:double) =
        Point2D(x,y)
```

## 抽象类（Abstract Class）和密封（Sealed）

要将类定义为抽象类或密封类，只需要在类定义前加上**`[<AbstractClass>]`**或**`[<Sealed>]`**特性。

### 抽象方法和虚方法

在F#中，没有提供`virtual`关键字来定义虚方法。而使用定义一个**抽象方法**，并提供**默认实现**来代替。

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

与C#一样，在派生类中进行**override**关键字重写基类中的虚方法：

```F#
type Point2D(xValue:double, yValue:double) =
    ……
    //重写Object类中的ToString虚方法，此处省略其他代码
    override this.ToString() = sprintf "Point2D(%f, %f)" this.X this.Y
```

在C#中，若要重写非虚方法，可使用`new`关键字。F#中没有此关键字，但仍然可以重写非虚方法，只是编译器会给出警告。*类的继承与接口的实现在下一篇中介绍。*

## 索引器（Indexer）及切片（Slice）

### 索引器

索引器，顾名思义就是可以使用索引来操作对象。在F#中，只需定义一个名为`Item`的属性或方法即可让该类的对象使用索引器。当然，若`Item`定义为方法，则索引器为只读的。

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

### 切片

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

此切片方法并未实现完整，但已可了解实现方法。注意`GetSlice`的参数必须是`'a option`泛型类型， 其中`option`类型将在后面再介绍，可理解为类似于`Nullable<int>`。

## 结语

通过以上的介绍，熟悉C#面向对象的朋友内容应该能了解F#与C#间语法的差别了。至少也可以将C#代码按面向对象的风格翻译成F#代码了，下面是文章开头C#代码的F#版本：

```F#
open System.Diagnostics
[<DebuggerDisplay("({X}, {Y})")>]
type Point2D (xValue:double, yValue:double) as self =
    let mutable x = xValue
    static let mutable count = 0    
    do        
        count <- count + 1
    
    member __.OriginalPoint with get () = Point2D()
    member this.LengthToOriginal 
        with get () = this.GetDistance(this.OriginalPoint)
        
    new() = Point2D(0.0,0.0)
    
    member __.X with        get()  = x
                and private set(v) = x <- v
    member val Y = yValue with get, set
    
    // 一个虚函数：获取指定点到当前点的距离
    abstract GetDistance : point:Point2D -> double
    default this.GetDistance(point) =
        let xDif = this.X - point.X
        let yDif = this.Y - point.Y
        let distance = sqrt (xDif**2. + yDif**2.)
        distance
        
    static member FromXY (x:double, y:double) =
        Point2D(x,y)
        
    override this.ToString() = sprintf "Point2D(%f, %f)" this.X this.Y
```

需要注意的是C#代码中的`OriginalPoint`公开字段在此处以只读属性实现。主要是因为：

- F#中的`let`绑定的字段只能为`private`，无法设置为public。
- `val`绑定的显示字段（包括自动属性）需要在主构造函数中初始化，而`OriginalPoint`的类型也为`Point2D`，在此会形成递归调用而引发`StackOverflowException`异常。

最后再简单说下类中`let`和`val`的区别：

- `let`始终是`private`的，且需要有主构造函数才能定义，因为let语句在主构造函数中运行（同`do`语句）。
- `val`默认为`public`的，并且可添加修饰符使之成为`internal`或`private`的。若有主构造函数，需要为`val`字段添加`DefaultValue`特性。`val`与`member`关键字结合使用，可声明自动属性。
- 在类中访问`val`字段，需要使用对象标识符，而`let`字段不需要。

注意，此`DefaultValue`位于**`Microsoft.FSharp.Core`**命名空间，不要和C#中常用的位于`System.ComponentModel`的`DefaultValue`混淆。

---

*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 转载请注明源链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-6.html>。*










