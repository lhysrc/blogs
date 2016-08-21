# 函数

[TOC]



*本文链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-4.html>*  

函数式编程其实就是按照数学上的函数运算思想来实现计算机上的运算。虽然我们不需要深入了解数学函数的知识，但应该清楚函数式编程的基础是来自于数学。

例如数学函数$f(x) = x^2+x$，并没有指定返回值的类型，在数学函数中并不需要关心数值类型和返回值。F#代码为`let f x = x ** 2.0 + x`，F#代码和数学函数非常类似，其实这就是函数式编程的思想：**只考虑用什么进行计算以及计算的结果**（或者叫“输入和输出”），并不考虑怎样计算。

其实，你可以把任何程序看成是一系列函数，**输入**是你鼠标和键盘的操作，**输出**是程序的运行结果。你不需要关心程序是怎样运行的，这些函数会根据你的输入来输出结果，而其中的算法是以函数的形式，而不是类或者对象。

下面我们就先了解一些函数式编程中函数相关的东西。

## 了解函数

### 不可变性

在一个函数中改变了程序的状态（比如在文件中写入数据或者在内存中改变了全局变量）我们称为**副作用**。像我们使用`printfn`函数，无论输入是什么，返回值均为`unit`，但它的副作用是打印文字到屏幕上了。

副作用并不一定不好，但却经常是很多bug的根源。我们分别用命令式和函数式求一组数字的平方和：

```
let square x = x * x
let sum1 nums =
    let mutable total = 0
    for i in nums do 
        let x = square i
        total <- total + x
    total
let sum2 nums =
    Seq.sum (Seq.map square nums)
```

*在`sum2`中使用了Seq模块中的函数，这些函数将在稍候进行介绍。*

可以看出，函数式代码简短了许多，且少了很多变量的声明。而且`sum1`是顺序执行，若想以并行方式运行则需要更改所有代码，但`sum2`只需要替换其中的`Seq.sum`和`Seq.map`函数。

### 函数和值

在我们接触到的非函数式编程语言（包括C#）中，函数和数值总是有一些不同。但在函数式编程语言中，**函数也是值**。比如，函数可以作为其他函数的参数，也可以作为返回值（即**高阶函数**）。而这在函数式编程中是非常常见的。  
需要注意的是，我们叫“**值**”而不叫“**变量**”。因为在函数式编程中声明的东西默认是不可变的。*（在F#中不完全如此，是因为F#包含了面向对象编程范式，可以说并非纯函数式编程语言。）* 

我们看下面以**函数作为参数**的代码（求一组数字的负值）：

```
> let negate x = -x;;
val negate : x:int -> int
> List.map negate [1..5];;
val it : int list = [-1; -2; -3; -4; -5]
```

我们使用函数``negate``和列表`[1..5]`作为`List.map`的参数。

但很多时候我们不需要声名一个名数名，只需使用**匿名函数**或叫**Lambda表达式**。在F#中，Lambda表达式为：关键字`fun`和参数，加上箭头`->`和函数体。则上面的代码可以更改为：

```
List.map (fun i-> -i) [1..5];;
```

我们再看以**函数作为返回值**的例子，假设我们定义一个`powOf`函数，输入一个值，返回一个该值求幂的函数：

```
let powOf baseValue =
    (fun exp -> baseValue ** exp)

let powOf2 = powOf 2.0 	// f(x) = 2^x
let powOf3 = powOf 3.0	// f(x) = 3^x
powOf2 8.				// 256.0
powOf3 8.   			// 6561.0
```

其中`powOf2`即为`powOf`函数使用参数`2`返回的函数。*其实这里涉及到**闭包**的内容，就不详细解释了，我们详细函数式编程时可能会再提及。*

#### 递归

递归大家都熟悉，只是在F#中声明时，需要添加`rec`关键字：

```
let rec fact x =
    if x <= 1 then 1
    else x * fact (x-1)
fact 5;;
(*
	val fact : x:int -> int
	val it : int = 120
*)
```

其实需要显示声明递归是因为F#的类型推断系统无法在函数声明完成之前确定其类型，而使用`rec`关键字后，就允许在确定类型前调用该函数。

#### 部分函数：Partial Function

在函数式编程中，还有一个叫Partial Function（暂且叫部分函数吧）的，可以把接收多个参数的函数分解成接收单个参数，即**柯里化（Currying）**。

我们知道，使用函数`printfn`打印整数的语句为`printfn "%d" i`，我们定义一个打印整数的函数：

```
> let printInt i = printfn "%d" i;;
val printInt : i:int -> unit
> let printInt = printfn "%d";;
val printInt : (int -> unit)
```

## 符号函数

在F#中，如`+ - * /`等运算符其实属于内建函数。而我们也可以使用这些符号来自定义符号函数。

我们用符号来重新定义上面的阶乘函数：

```
let rec (!) x =
    if x <= 1 then 1
    else x * !(x - 1)
!5;;
(*
	val ( ! ) : int -> int
	val it : int = 120
*)
```

需要注意的是，符号函数一般需要括号包裹，如果符号函数的参数不止一个，则符号函数是以**中缀**的方式来使用，例如我们用`=~=`定义一个验证字符串是否和正则表达式匹配的函数：

```
open System.Text.RegularExpressions;;
let (=~=) str (regex : string) =
    Regex.Match(str, regex).Success
"The quick brown fox" =~= "The (.*) fox";;
(*
	val ( =~= ) : string -> string -> bool
	val it : bool = true
*)
```

而且，**符号函数也可以作为高阶函数的参数**。

### 管道符：`|>`和`<|`

我们再返回来看上面的平方和函数：

```
let sum2 nums =
    Seq.sum (Seq.map square nums)
```

假如函数层次非常多，一层包裹一层，则可读性非常差。

在F#定义了如下符号函数

```
let (|>) x f = f x
let (<|) f x = f x
```

我们称为“**正向管道符**”和“**逆向管道符**”。则上面的平方和函数可写作：

```
let sum2 nums = 
	nums 
	|> Seq.map square 
	|> Seq.sum
```

`<|`虽然用得不多，但大多用来改变优先级而无需使用括号：

```
let sum2 nums =
    Seq.sum <| Seq.map square nums
```

### 合成符：`>>`和`<<`

我们也可以用**函数合成符**将多个函数组合成一个函数，合成符也分正向（`>>`）和逆向（`<<`）。

```
let (>>) f g x = g(f x)
let (<<) f g x = f(g x)
```

还是以上面的求平方和为例：

```
let sum2 nums = (Seq.map square >> Seq.sum) nums
let sum2 nums = (Seq.sum << Seq.map square) nums
```

## 常用模块函数

在[上一篇](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-3.html)中，我们了解了集合类型。在F#中，为这些集合类型定义了许多函数，分别在集合名称对应的模块中，例如Seq的相关函数位于模块`Microsoft.FSharp.Collections.Seq`中。而这也是我们最常用到的模块函数。

下面简单介绍其中常用的函数，大家可通过MSDN了解更多：

[Seq模块](https://msdn.microsoft.com/visualfsharpdocs/conceptual/collections.seq-module-%5bfsharp%5d)、[List模块](https://msdn.microsoft.com/visualfsharpdocs/conceptual/collections.list-module-%5bfsharp%5d)、[Array模块](https://msdn.microsoft.com/visualfsharpdocs/conceptual/collections.array-module-%5bfsharp%5d)。











本文链接：[http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-4.html](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-4.html)  

