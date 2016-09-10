

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

```
type Point2D() =
    [<DefaultValue>] val mutable public x : float
```

其中`DefaultValue`特性用于将支持零值初始化的类型（具有零值的基元类型、可为null的类等）字段初始化为零值。

### 属性（Property）和索引器

使用`member`关键字定义属性，注意F#中的属性需要有一个自我标识符（类似C#中的`this`）作为前缀。

```
type Point2D() =
    let mutable x = 0.0
    // 定义属性X，其中set为private的。
    member this.X with get()  = x
                  and private set(v) = x <- v
    member val Y = 0.0 with get, set	//自动属性
```

在F#中，`this`也可以使用其他名称来代替。除了`this`（C++和C#的习惯），有的人还使用`self`（Python等的习惯）、`me`（VB.NET的习惯）等。若不需要在属性或方法内部使用到，一般还习惯使用`__`（两个下划线）来作为自我标识符。

而像C#`set`方法中的`value`，F#中也需要自己指定，如例子中使用`v`。当然`get`和`set`都可以省略使之成为只写或只读属性。

遗憾的是自动属性中不支持分别定义访问修饰符（如C#中的`{get; private set}`），只能使用完整属性来定义。

### 方法（Method）

方法同样使用`member`定义，而静态方法只需在`member`前添加`static`关键字。

```
// define a Point2D class
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



### 索引器及切片



## 继承和扩展

### 类型转换

### 泛型及约束



### 扩展

#### 类级扩展

#### 对象级扩展




## 接口



##结构和枚举





























-   类扩展方法
-   对象级扩展方法(F#程序设计P174)    
-   结构 

