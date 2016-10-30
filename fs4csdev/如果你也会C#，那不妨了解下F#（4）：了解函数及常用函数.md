# 了解函数及常用函数

[TOC]
## 前言
函数式编程其实就是按照数学上的函数运算思想来实现计算机上的运算。虽然我们不需要深入了解数学函数的知识，但应该清楚函数式编程的基础是来自于数学。

例如数学函数$f(x) = x^2+x$，并没有指定返回值的类型，在数学函数中并不需要关心数值类型和返回值。F#代码为`let f x = x ** 2.0 + x`，F#代码和数学函数非常类似，其实这就是函数式编程的思想：**只考虑用什么进行计算以及计算的结果**（或者叫“输入和输出”），并不考虑怎样计算。

其实，你可以把任何程序看成是一系列函数，**输入**是你鼠标和键盘的操作，**输出**是程序的运行结果。你不需要关心程序是怎样运行的，这些函数会根据你的输入来输出结果，而其中的算法是以函数的形式，而不是类或者对象。

下面我们就先了解一些函数式编程中函数相关的东西。

## 了解函数

### 不可变性

在一个函数中改变了程序的状态（比如在文件中写入数据或者在内存中改变了全局变量）我们称为**副作用**。像我们使用`printfn`函数，无论输入是什么，返回值均为`unit`，但它的副作用是打印文字到屏幕上了。

副作用并不一定不好，但却经常是很多bug的根源。我们分别用命令式和函数式求一组数字的平方和：

```F#
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

```F#
> let negate x = -x;;
val negate : x:int -> int
> List.map negate [1..5];;
val it : int list = [-1; -2; -3; -4; -5]
```

我们使用函数``negate``和列表`[1..5]`作为`List.map`的参数。

但很多时候我们不需要给函数一个名称，只需使用**匿名函数**或叫**Lambda表达式**。在F#中，Lambda表达式为：关键字`fun`和参数，加上箭头`->`和函数体。则上面的代码可以更改为：

```F#
List.map (fun i-> -i) [1..5];;
```

我们再看以**函数作为返回值**的例子，假设我们定义一个`powOf`函数，输入一个值，返回一个该值求幂的函数：

```F#
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

```F#
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

```F#
> let printInt i = printfn "%d" i;;
val printInt : i:int -> unit
> let printInt = printfn "%d";;
val printInt : (int -> unit)
```

## 符号函数

在F#中，如`+ - * /`等运算符其实属于内建函数。而我们也可以使用这些符号来自定义符号函数。

我们用符号来重新定义上面的阶乘函数：

```F#
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

```F#
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

```F#
let sum2 nums =
    Seq.sum (Seq.map square nums)
```

假如函数层次非常多，一层包裹一层，则可读性非常差。

在F#定义了如下符号函数

```F#
let (|>) x f = f x
let (<|) f x = f x
```

我们称为“**正向管道符**”和“**逆向管道符**”。则上面的平方和函数可写作：

```F#
let sum2 nums = 
	nums 
	|> Seq.map square 
	|> Seq.sum
```

`<|`虽然用得不多，但常用来改变优先级而无需使用括号：

```F#
let sum2 nums =
    Seq.sum <| Seq.map square nums
```

### 合成符：`>>`和`<<`

我们也可以用**函数合成符**将多个函数组合成一个函数，合成符也分正向（`>>`）和逆向（`<<`）。

```F#
let (>>) f g x = g(f x)
let (<<) f g x = f(g x)
```

还是以上面的求平方和为例（`Seq.map square`即是一个**部分函数**）：

```F#
let sum2 nums = (Seq.map square >> Seq.sum) nums
let sum2 nums = (Seq.sum << Seq.map square) nums
```

## 常用模块函数

在[上一篇](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-3.html)中，我们了解了集合类型。在F#中，为这些集合类型定义了许多函数，分别在集合名称对应的模块中，例如Seq的相关函数位于模块`Microsoft.FSharp.Collections.Seq`中。而这也是我们最常用到的模块。

**模块（`module`）**是F#中组织代码的一种方式，类似于命令空间（`namespace`）。但F#中也是有命名空间的，其间的区别将在下一篇介绍。

下面简单介绍常用的函数，并会列出与.Net的`System.Linq`中对应的函数。

如无特别说明，该函数在三个模块中均可用，但**因为集合的实现方式不同，函数的复杂度也会有区别**，在使用中根据实际情况选择合适的函数。

#### length

对应于Linq中的`Count`。即获得集合中**元素的个数**。

```F#
[1..10] |> List.length;;	// 10
Seq.length {1..100};;		// 100
```

*虽然在`Seq`中也有`length`函数，但谨慎使用，因为Seq可能为无限序列。*

#### exists 和 exists2

`exists`用于判断集合是否存在符合给定条件的元素，对应于Linq中的`Any`。而`exists2`用于判断**两个集合**是否包含在**同一位置**且符合给定条件的一对元素。

```F#
List.exists ((=) 3) [1;3;5;7];;		//true
Seq.exists (fun n1 n2 -> n1=n2) {1..5} {5..-1..1};;	//true
```

第一行代码判断列表中是否包含等于3的元素，其中`(=) 3`即为**部分函数**，注意`=`为符号函数。

第二行代码判断两个序列中，因为`{1;2;3;4;5}`和`{5;4;3;2;1}`在索引2的位置存在元素符合函数`(fun n1 n2 -> n1=n2)`，所以返回`true`。

#### forall 和 forall2

`forall`检查是否集合中**所有**元素均满足指定条件，对应Linq中的`All`。

```F#
let nums = {2..2..10}
nums |> Seq.forall (fun n -> n % 2 = 0);;	//true
```

而`forall2`和`exists2`类似，但当且仅当**所有元素**都满足相同位置且符合给定条件才返回`true`。接上一个代码片段：

```F#
let nums2 = {12..2..20}
Seq.forall2 (fun n n2 -> n + 10 = n2) nums nums2;;	//true
```

#### find 和 findIndex

`find`查找符合条件的**第一个元素**，对应Linq中的`First`。需要注意的是当不存在符合条件的元素，将引发`KeyNotFoundException`异常。

```F#
Seq.find (fun i -> i % 5 = 0) {1..100};;	//5
```

`findIndex`则返回符合条件的第一个元素的**索引**。

#### map 和 mapi

`map`对应Linq中的`Select`，将函数应用于集合中的每个元素，返回值产生一个新的集合。

```F#
List.map ((*) 2) [1..10];;	
// [2; 4; 6; 8; 10; 12; 14; 16; 18; 20]
```

`mapi`与`map`类似，不过在应用的函数中还需要传入一个整数作为集合的索引。
```F#
Seq.mapi(fun i x -> x*i) [3;5;7;8;0];; 
// 将各个元素乘以各自的索引，结果为：[0; 5; 14; 24; 0]
```

#### iter 和 iteri

`iter`将函数应用于集合中的每个元素，但函数返回值为unit。功能类似于`for`循环。
而`iteri`与`mapi`一样需要在函数中传入一个索引。
```F#
Seq.iteri(fun i x -> printfn "第%d个元素为：%d" i x) [3;5;7;8;0]
(*
    第0个元素为：3
    第1个元素为：5
    ……
*)
```

#### filter 和 where

F#中`filter`和`where`是一样的，对应于Linq中的`Where`。用于查找符合条件的元素。

```F#
{1..10} |> Seq.filter (fun n -> n%2 = 0);;
//val it : seq<int> = seq [2; 4; 6; 8; ...]
```

#### fold

`fold`对应Linq中的`Aggregate`，通过提供初始值，然后将函数逐个应用于每个元素，返回单一值。

```F#
Seq.fold (fun acc n -> acc + n) 0 {1..5};;	//15
Seq.fold (fun acc n -> acc + string n) "" {1..10};;	
//"12345"
```

首先，将初始值与第一个元素应用于函数，再将返回值与第二个元素应用于函数，依此类推……

Linq中的``Aggregate``包含不需要提供初始值的重载，其实F#中也有对应的`reduce`函数。类似的还有`foldBack`和`reduceBack`等逆向操作，这里就不介绍了。

#### collect

`collect`对应Linq中的`SelectMany`，展开集合并返回所有**二级集合**的元素。

```F#
let lists = [ [0;1]; [0;1;2]; [0;1;2;3] ]
lists |> List.collect id;;
//[0; 1; 0; 1; 2; 0; 1; 2; 3]
```

其中**id**为`Operators`模块中的函数，它的实现为`fun n->n`，即直接对参数进行返回。

#### append

`append`将两个集合类型合并成一个，对应于Linq中的`Concat`。

```F#
> Array.append [|1;3;1;4|] [|5;2;0|];;
val it : int [] = [|1; 3; 1; 4; 5; 2; 0|]
```

#### zip 和 zip3

`zip`函数将两个集合合并到一个里，合并后每个元素是一个二元元组。

```F#
let list1 = [ 1..3 ]
let list2 = [ "a";"b";"c" ]
List.zip list1 list2;;
// [(1, "a"); (2, "b"); (3, "c")]
```

`zip3`顾名思义，就是将三个集合合并到一个里。

**合并后的长度取决于最短的集合的长度。**

#### rev

`rev`函数反转一个列表或数组，在Seq模块中**没有**这个函数。

#### sort

`sort`函数基于compare函数（[第二篇](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-2.html)中的“比较”介绍过）对集合中的元素进行排序。

```F#
> List.sort [1;3;-2;2];;
val it : int list = [-2; 1; 2; 3]
```

### 数学函数

Linq中包含`Max`、`Min`、`Average`和`Sum`等数学函数。F#集合模块中也有对应的函数。

```F#
List.max [1..10]		//10
Seq.min {1..5}			//5
[1..10] |> List.map float |> List.average	//5.5
List.averageBy float [1..10]				//5.5

[0..100] |> Seq.where (fun x -> x % 2 <> 0) |> Seq.sum |> printf "0到100中的奇数的和为%i"
// 0到100中的奇数的和为2500
```

需要注意的是，`average`函数需要集合中的元素支持**精确除法**（Exact division，即实现了`DivideByInt`函数的类型。*不知道为什么是ByInt。*），而F#中又**不支持隐式类型转换**，所以对`int`集合求平均值只能先转换为`float`或`float32`，或使用`averageBy`函数。

`sum`函数的示例代码将[第一篇](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-1.html)中由C#翻译过来的命令示示例代码转换成了函数式的代码。

### 集合间转换

三种集合类型的对应模块中，均提供转换**到（to）**另外两种集合类型，和**从（of）**另外两种类型转换的函数。

如Seq模块，通过`Seq.toList`和`Seq.toArray`函数转出；通过`Seq.ofList`和`Seq.ofArray`转入。

```F#
Seq.toList {1..5};;			//[1; 2; 3; 4; 5]
List.ofArray [|1..5|];;		//[1; 2; 3; 4; 5]
```

---

函数式编程，核心就是函数的运用。上面介绍的这些在C#中也经常使用到对应的方法，但F#提供的函数非常丰富，大家可通过MSDN了解更多：

-   [Seq模块](https://msdn.microsoft.com/visualfsharpdocs/conceptual/collections.seq-module-%5bfsharp%5d)  
-   [List模块](https://msdn.microsoft.com/visualfsharpdocs/conceptual/collections.list-module-%5bfsharp%5d)
-   [Array模块](https://msdn.microsoft.com/visualfsharpdocs/conceptual/collections.array-module-%5bfsharp%5d)

因为F#中的List和Array均实现了`IEnumarable<T>`接口，所以**Seq模块的函数也可以接收List类型和Array类型的参数**。当然，反之则不行。

到现在为止，我们了解的F#都是在交互窗口中。下一篇我们再简单介绍项目创建和代码组织，即**模块**相关。

*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 原文链接为：[http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-4.html](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-4.html)。*