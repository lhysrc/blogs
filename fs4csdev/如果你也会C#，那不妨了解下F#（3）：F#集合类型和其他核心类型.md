# F#集合类型和其他核心类型
[TOC]

##前言
在第一篇中，我们介绍了一些基础数据类型，其实那篇标题中不应该含有“F#”字眼，因为并不是特有的。
在本篇中，我们介绍如数组这些集合类型以及部分F#特有的类型。


在第一篇里我们列了一个从0加到100的代码段，了解函数式编程的同学会说那个F#代码不正宗。
而现在的C#开发一般也会使用Linq的方式来代替循环，其实F#天生就是使用这种方式的，下面我们先介绍F#的集合类型，在之后会介绍相关集合的函数操作。

## 集合类型

### 列表：List

#### 声明List

**注意，F#中的List不是C#中常用的`System.Collections.Generic.List<T>`**，虽然后者你也可以在F#里使用。

F#中的List是**有序，不可变的，且每一项的类型必须一致**。我们先看看怎么定义列表。*以下代码以`>`开头的为输入，之后的一行则为输出结果。*

```typescript
> let charList = ['a';'o';'e';'i';'u';'ü'];; 
val charList : char list = ['a'; 'o'; 'e'; 'i'; 'u'; 'ü']
> let emptyList = [];;
val emptyList : 'a list
> let emptyList2 = List.empty;;
val emptyList : 'a list
```

使用`[]`声明和List.empty均可声明空列表，空列表类型为`'a list`表示可接收任意类型，但**当你添加一个元素后，列表类型但确定了，无法添加其它类型的元素。**

和C#不一样，F#在集合中使用分号（`;`）分隔各个项。但当列表很大时，使用这样的声明方式就变得很麻烦了。

这时，可以使用下面的范围（Range）声明方式来声明：

```F#
> let intList = [1..10];;
val intList : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
> let tens = [0 .. 10 .. 100];;
val tens : int list = [0; 10; 20; 30; 40; 50; 60; 70; 80; 90; 100]
> let countDown = [5L .. -1L .. 0L];;
val countDown : int64 list = [5L; 4L; 3L; 2L; 1L; 0L]
```

范围声明方式中，中间项为间隔（若无则间隔为1），并**包含起始项到结束项**。*熟悉Python的小伙伴要注意。*😁

除了范围式声明，还可以使用推导式声明：

```F#
> let intList2 = [for i=0 to 8 do yield i*i];;
val intList2 : int list = [0; 1; 4; 9; 16; 25; 36; 49; 64]
```

推导式声明代码使用`[]`包裹，通过执行里面的代码并在每次执行到`yield`语句时生成一个元素。

**注意：F#的List中`yield`生成元素是一次性完全在内存中生成，若元素太多，应使用序列（`seq`）。** *将在后面介绍。*

当然，由`[]`包裹的声明式代码可以是复杂的语句，请看下面生成质数列表的语句：

```F#
> let primesUnder50 =
    [
        for n in 1 .. 50 do
            let factorsOfN =
                [
                    for i in 1 .. n do
                        if n % i = 0 then
                            yield i
                ]
            // 如果一个数的因数只有1和自己，便是质数
            if List.length factorsOfN = 2 then
                yield n
    ];;
val primesUnder50 : int list = [2; 3; 5; 7; 11; 13; 17; 19; 23; 29; 31; 37; 41; 43; 47]
```

经过前两章介绍，应该可以发现，F#不使用大括号包裹代码块，而像Python使用**缩进**。这段代码我们就先不解释了。

#### List操作

-   索引取值

    在F#中，使用索引进行取值和方法一样也需要用`.`标记。

    ```F#
    list0[0] //错误
    list0.[0] //正确
    ```

-   比较

    如果两个列表类型相同，且类型可以进行比较（即实现`IComparable`接口），则可进行比较操作：

    ```F#
    let list1= [1;3;4;5]
    let list2= [1;4;3]
    list1 > list2;; //结果为false
    ```

-   `::`（cons）和`@`操作符

    **``::``** 将一个元素添加到列表的开头，而`@`则将两个列表相连。例如`list1`为`[2;3;4]`：

    ```F#
    let list2 = 100 :: list1 ;; // [100; 2; 3; 4]
    let list2 = list1 @ list2 ;; // [2; 3; 4; 100; 2; 3; 4]
    ```

    因为列表为不可变类型，所以操作后均返回了一个新的列表。但在内部实现上，`::`只需将新项连接到原列表（O(1)），而`@`操作需克隆第一个列表，所以**`::`总是优于`@`操作**。这方面有兴趣可查看F#的List实现。

#### List属性

| Property                                 | Type      | Description     |
| ---------------------------------------- | --------- | --------------- |
| [Head](https://msdn.microsoft.com/library/5f9414fd-6bdb-470a-8b72-40016db30740) | `'T`      | 返回第一项。          |
| [Empty](https://msdn.microsoft.com/library/44406ecb-1918-4d32-b32a-ca1f69840386) | `'T list` | **静态属性**，返回空列表。 |
| [IsEmpty](https://msdn.microsoft.com/library/3ba087b2-2fc2-406d-b10a-cff6a19322da) | `bool`    | 判断是否为空。         |
| [Item](https://msdn.microsoft.com/library/bdb2553a-0e54-4ff8-baed-ab1aac8f5dae) | `'T`      | 返回指定项。          |
| [Length](https://msdn.microsoft.com/library/25f715c8-9daa-4c4d-a6c7-26772f9dab4d) | `int`     | 列表包含元素个数。       |
| [Tail](https://msdn.microsoft.com/library/2a6f8eb9-dc32-41aa-8b62-2baffaface91) | `'T list` | 返回除第一个元素外的其他元素。 |

```F#
> [2;3;4;5;6;6].Tail;;
val it : int list = [3; 4; 5; 6; 6]
```

F#的List在.Net框架里的类型为`Microsoft.FSharp.Collections.FSharpList<T>`，若想在C#中使用，需要添加`FSharp.Core.dll`引用。关于List的其他信息可查阅MSDN文章[Lists (F#)](https://msdn.microsoft.com/en-us/visualfsharpdocs/conceptual/lists-%5Bfsharp%5D)。而List模块相关函数将在函数式编程时再细说。

### 序列：Seq

F#中的`seq<_>`其实就是.Net中的`IEnumerable<T>`，声明时只需将List声明中的`[]`换成`{}`即可。

```F#
> let intSeq = {1..10};;
val intSeq : seq<int>
> let intSeq2 = seq {for i=0 to 8 do yield i*i};;
val intSeq2 : seq<int>
```

在推导式声明中，需要在`{}`前加**`seq`**关键字。

``IEnumerable``是在需要时才对其中的元素进行计算（**惰性计算**），所以在输出结果时并不显示具体元素。*适用于具有大量元素的集合。*所以在声明时，无法使用`seq { 1; 2; 3 }`来声明，需要使用`seq { yield 1; yield 2; yield 3 }`。也不支持索引操作。

`IEnumerable<T>` 大家都很熟悉，在下就不班门弄斧了。🙁

###  数组：Array

数组是固定**长度、可变、每项类型一致**的集合。F#中的数组和C#的一样，但声明方式是类似于F#中的`list`，使用`[| |]`包裹。但数组的**类型名称是`[]`**，例如：

```F#
> let squares1 = [| for i in 1 .. 7 -> i * i |];;		//推导式声明
val squares1 : int [] = [|1; 4; 9; 16; 25; 36; 49|]
> let squares2 = [| 1; 4; 9; 16; 25; 36; 49; 64; 81 |];;		//手动声明
val squares2 : int []
> squares1.[5];;
val it : int = 36
```

数组支持索引取值，也是使用`.`加`[index]`的方式。数组间也支持比较。

#### 数组切片：Slices

在F#中，数组可通过切片操作取得一部分连续的元素，语法是`[start..end]`，可省略。接上一个例子：

```F#
> suares1.[1..4];;
val it : int [] = [|4; 9; 16; 25|]
> suares1.[1..];;	//省略结束项，则取到最后一项
val it : int [] = [|4; 9; 16; 25; 36; 49|]
> suares1.[..4];;	//省略起始项，则从第一项开始
val it : int [] = [|1; 4; 9; 16; 25|]
> suares1.[*];;					
val it : int [] = [|1; 4; 9; 16; 25; 36; 49|]
```

`*`则可用来复制数组。**`list`可支持切片操作，有兴趣的同学可自行尝试。** *不像Python，F#的切片中不支持间隔。*

### 可变集合类型

在F#中，使用`System.Collections.Generic`下的其他集合类型也是很常见的。这些类型大家都很熟悉了，不再赘述。

列些代码供大家参考：

```F#
// List<'T>
let animails = System.Collections.Generic.List<string>()
animails.AddRange( [| "猪"; "羊"; "鸭"; "狗"; "猫" |] )
animails.Remove("鸭")
animails.Count;;
(*
    val animails : List<string>
    val it : int = 4
*)
```

```F#
open System.Collections.Generic
let dict = new Dictionary<string, string>()
dict.Add("A","Atom")
dict.Add("B","Bob")
let atom = dict.["A"]
let found, bob = periodicTable.TryGetValue("B");;
(*
    val dict : Dictionary<string,string> = dict [("A", "Atom"); ("B", "Bob")]
    val atom : string = "Atom"
    val found : bool = false
    val bob : string = null
*)
```

*还有其他如`Hash<T>`等类型不再举例了。*

从上面的代码可以看出，类在初始化时可以使用`new`；打开模块或命名空间使用`open`；方法参数中有`out`关键字的，F#中使用多返回值返回（即`Tuple`类型）。这些将在之后F#的面向对象编程中涉及。



## 其他F#类型

### 常量

C#中使用`const`定义的常量，在F#中需要使用`Literal`特性（Attribute）定义。

```F#
[<Literal>]
let MyConstant = 99
```

在F#中，定义数据类型时使用**特性**多于使用关键字。而F#中的**特性需要放于`[< >]`内**。

### 元组：Tuple

元组是一组有序的项，其中可以包含不同类型。

```F#
(1,5);;		// val it : int * int = (1, 5)
("xyz",'c',2.0+3.);;	// val it : string * char * float = ("xyz", 'c', 5.0)
```

注意元组的类型名称为`'T * 'T ...`，以`*`间隔其中的不同项。

### 函数

函数在F#中也可看作是种类型，定义函数也很简单，和C#类似。

但需要注意的是，F#中**不使用`return`关键字，而使用最后一条语句作为返回值**。

```F#
let addOne x = x + 1;; // val addOne : int -> int
let sum x y = x + y;; // val sum : int -> int -> int
```

定义一个`addOne`函数，以输入参数和输出参数均为`int`。

F#中函数参数不需要使用`()`包裹，而只需跟在函数名称之后。而使用`(x,y)`在F#中将被看作为一个元组参数。

*函数先不介绍太多，将在下一篇中详细介绍。*

### Unit

在C#中，函数若没有返回值，使用`void`声明。在F#中，使用`unit`。若要定义一个Unit类型，使用`()`。

```F#
let f (i:int) = ()
```

定义一个输入为`int`而无返回值的函数。

### 类型别名

在F#中可给类型设置一个别名，如给`int`一个其他的名称，类似于C++中的`typedef`。

```F#
typle I = int
let fn (a:I) = a + 1;;	// val fn : a:I -> I
```

介绍`seq`时，说``seq<_>``其实就是.Net中的``IEnumerable<T>``，查看源代码，我们可以发现：

```F#
type seq<'T> = System.Collections.Generic.IEnumerable<'T>
```



*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 转载请注明源链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-3.html>。*

