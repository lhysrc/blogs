# 代码组织、与C#互相调用

[TOC]

## F# 项目

在之前的几篇文章介绍的代码都在交互窗口（fsi.exe）里运行，但平常开发的软件程序可能含有大类类型和函数定义，代码不可能都在一个文件里。下面我们来看VS里提供的F#项目模板。

F#项目模板有以下几种类型（以VS2015为例）： ![F#项目模板](https://qtjwpa.dm2302.livefilestore.com/y3m-gh9vFapjy3dCC3IwQIXCsduO3UwcgsFAJT42gV8mH4JwrtEEVyy0xJ3OoGkSs70igKP-wZ0qlOxJw08th4OykH-UVb4nVw4fDE6cp4KpXhG8k7CzspAvx66MgeH6HFriEW1S80g0CMwyMSdy92LHMM9hkeORSkxfLNFHBZ8ryM?width=942&height=517&cropmode=none)

-   Silverlight库创建Silverlight的类库
-   教程模板是一个控制台应用程序，里面包含了F#的示例，可通过这个项目快速了解F#相关内容。
-   “可移植库”则可创建用于多平台的库，支持的平台在括号里说明。
-   “**库**”用于创建类库
-   “**控制台应用程序**”大家就熟悉了。
-   *安卓项目为安装了Xamarin创建的，请忽略。*

我们创建一个控制台应用程序来说明，下图为程序的`Program.fs`文件及运行结果：

![F#控制台程序及运行结果](https://qtjxpa.dm2302.livefilestore.com/y3mcpJ7Ax7cvJVXPe8HY9Q0T7eVH9tVK7QZ9hB6LMLltKV58nSRLAPtcliXrA3UBg3PmiRLk0s89j_b1AtjGTPvk9sElZZsSCTas9Mt0U7SmxsTuBeXR0YWhKekNlc1NU95sEzL9fpc0lOI7cs6Puwxx0PFkmYQspv6uolwTvJvIK0?width=563&height=366&cropmode=none)

我们添加一行代码（图中蓝框中）防止运行结束自动退出，这个应用程序默认是把参数打印出来，而运行时参数为空，所以结果为一空数组（`[||]`）。

其中**`ignore`函数用于丢弃`System.Console.ReadKey()`结果**。

现在项目中除了`AssemblyInfo.fs`外，只有`Program.fs`一个文件，下面我们先了解**模块**的相关信息再创建其他文件。

## 模块

### 模块简介

**模块（Module）**是F#程序代码的基本组织单位。默认情况下，每个F#代码文件（后缀为`.fs`）对应一个模块，且必须在文件开头指定模块名称。

#### 创建模块

我们创建`File1.fs`文件时，默认会在开头添加`module File1`，当然也可自己改成其他名称。

```F#
module File1
let x = 1
```

在其他模块中使用`File1.x`进行访问。

#### 文件顺序

**F#项目中的文件是有顺序要求**的，在上面的文件无法访问下面的模块。我们可以使用**Alt+上/下箭头**进行调整文件顺序，或在文件上点击右键进行操作：  ![F#代码顺序](https://qjjopa.dm2302.livefilestore.com/y3mCfNebtImIBcoRkWuJTLgpJWsB1QmdprcOeSTq8etEBYiLXReF_2I1mPKJ0cICWj3d_a0N5ZwQj5wehegp3l5YbIif0Yr6cica_xPhOcaQn4tJwnTZYLwfWbFlUCRI8TASmYtK-DvUKOsxQ55cGmHW2ETPBZF1fdhGcMtev8Asz4?width=484&height=311&cropmode=none)

#### 嵌套模块

模块中可嵌套模块，但定义内层模块需要在模块名后使用等号（`=`），且内层模块的内容必须比它的上层模块**缩进**一级。

```F#
module TopLevelModel		
module NestedModule = 	//第一层嵌套模块
    let i = 1
    module NestedModuleInNestedModule =  //第二层嵌套模块
        let i = 2
```

###  使用模块

若想不使用模块名访问模块中的值时，则可使用`open`关键字进行打开。但有两个需要注意的地方：

#### 强制显示访问

在上一章介绍的集合模块中，我们从未使用`open List`或者`open Seq`这样的操作。

使用F12转到`Seq`的代码定义文件可以发现`Seq`模块使用了
**`[<RequireQualifiedAccess>]`（强制显示访问）**。

附加了此特性的模块在使用时**必须使用模块名访问**，因为几个集合模块中有大部分函数名称是相同的，若设置此特性而可同时打开了多个模块，则函数名称将会冲突。

#### 自动打开

而我们在使用`printfn`和`ignore`函数时，均不需要打开相关模块，是因为在他们所属模块附加了**`[<AutoOpen>]`（自动打开）**的特性。像`Operators`模块里有我们常用的运算符，为了方便使用，添加了自动打开的特性。

我们在自定义模块时可根据需要使用这两个特性。

### 命名空间

命名空间（Namespace）和模块类似，不同的是**命名空间里不能直接定义值**，只能定义类型。（与C#中的命名空间一样，可以想象我们无法在C#的命名空间中直接定义一个方法，而需要首先定义一个类。）

但F#中的命名空间不能像模块那样嵌套，但可以在同一文件中定义多个命名空间。

```F#
namespace PlayingCards
type Suit = Spade | Club | Diamond | Heart

namespace PlayingCards.Poker
type PokerPlayer = {Name:string; Money:int; Position:int}
```

上面的代码在一个文件中使用两个命名空间分别定义了一个类型。

*其中`Suit`为**可区分联合（Discriminated Union）**类型；`PokerPlayer`为**记录（Record）**类型。将在下一篇介绍。*

### 应用程序入口

在F#中，程序从程序集的最后一个文件开始执行，而且必须是一个模块。但**最后一个模块的名称可省略**。

也可以使用`[<EntryPoint>]`特性应用于**最后一个代码文件的最后一个函数**，使其成为程序入口点而无需显示调用。

可查看控制台应用程序项目的模板：

```F#
[<EntryPoint>]
let main argv =     
    printfn "%A" argv
    0
```

`main`函数的参数是一个数组（通常可自定义为**字符串数组**），是应用程序的运行参数，返回的整数则为程序的**退出代码**（exit code）。

若不使用`[<EntryPoint>]`，则需要在最后调用该函数，否则并不会自动调用该函数。

```F#
let main (argv:string[]) = 
    printfn "%A" argv
    System.Console.ReadKey(true) |> ignore
    0
main [||]
```

*控制台应用程序通常在结束之前使用`System.Console.ReadKey()`方法来防止运行完成自动退出。*

### 扩展模块

可以通过创建一个同名模块，在其中添加值来对原有模块进行扩展。

在介绍常用函数时，我们提到`Seq`模块没有提供`rev`函数，现在自己实现以**对`Seq`模块进行扩展**。

```F#
open System.Collections.Generic
module Seq =
    /// 反转Seq中的元素
    let rec rev (s : seq<'a>) =
        let stack = new Stack<'a>()
        s |> Seq.iter stack.Push
        seq {
            while stack.Count > 0 do
                yield stack.Pop()
        }
```

其中使用了.NET框架中的泛型**栈**集合类型（`System.Collections.Generic.Stack<T>`）。

## 与C#互相调用

F#代码和C#代码（包括VB.NET）一样，都编译成MSIL，在CLR运行。（可参考文章[《.NET框架》](http://www.tracefact.net/CLR-and-Framework/DotNet-Framework.aspx)）所以，两种语言之间可以方便地互相调用。

程序集的引用大家都熟悉，但C#和F#中又有一些独立的东西不能互相使用，下面简单介绍一下在互相调用中常见的问题。

### F#调用C#代码

本节涉及操作需要创建两个项目，一个**C#的类库项目**，一个**F#的控制台项目**。然后F#项目引用C#项目。

#### dynamic：在F#中访问C#的动态类型

在.NET4.0，C#引入了`dynamic`关键字使得可以像使用动态语言一样来使用C#。但在F#中并不支持`dynamic`关键字和动态类型，在引用C#编译的程序集时，则变成了`Object`类型。

我们知道`dynamic`在`Microsoft.CSharp.dll`程序集中实现，在F#中可以通过引用此程序集，通过反射等操作自己实现对动态类型及属性的访问。

而我在平常一般使用第三方库`FSharp.Interop.Dynamic`([Nuget](https://www.nuget.org/packages/FSharp.Interop.Dynamic/))。代码示例：

```c#
//C#代码，命名空间CSharpForFSharp
public class CSharpClass
{
  public dynamic TestDynamic()
  {
    return "5566";
  }
}
```

在F#中调用：

```F#
//F#代码，位于F#项目的Program.fs
open FSharp.Interop.Dynamic
open CSharpForFSharp			//C#项目中的命名空间
[<EntryPoint>]
let main argv =     
    let cc = CSharpClass()
    let str = cc.TestDynamic()    
    printfn "%A" (str?Length)	//使用?替代.
    System.Console.Read()|>ignore
    0
```

打开`FSharp.Interop.Dynamic`命名空间，F#中可使用`?`来访问动态类型的属性和方法。 

#### 调用带有 `ref` 和 `out` 参数的函数

在C#中，有`ref`和`out`两个关键字来修饰函数的参数，使函数可以进行引用传递和返回多个值。若要在F#中调用，则有一些不同。

带有**`ref`**参数或者**`out`**参数的函数，因为参数值可能在函数中发生改变，需要在F#先定义一个**可变值类型**，并使用**寻址操作符（`&`）**进行传入。

```c#
// C#代码，位于命名空间CSharpForFSharp
public class CSharpClass
{
    public static bool OutRefParams(out int x, ref int y)
    {
        x = 100;
        y = y * y;
        return true;
    }
}
```
在F#中调用：

```F#
// F#代码，位于F#项目的Program.fs
open CSharpForFSharp
let mutable x,y = 0,0
CSharpClass.OutRefParams(&x,&y)	
```

返回`true`并对`x`和`y`进行了改变。

带有**`out`**的参数在C#中可以使用未赋值的变量传入，所以在F#中除了寻址传入的方法，还可以直接**忽略该参数**，则该函数在F#中成为了多返回值（即**返回`tuple`**）的形式：

```F#
let successful, result = Int32.TryParse(str)
```

`Int32.TryParse`返回了两个值，第一个总是函数返回值，而后是`out`参数。

#### 柯里化C#的方法

因为C#中的函数无论有多少个参数，在F#中调用时都视为**一个`tuple`参数**，所以无法柯里化和使用函数管道符（`|>`）操作。

在F#中可以使用`FuncConvert`类将.NET中的函数转换成F#中的函数。

```F#
let join : string*string list -> string = System.String.Join
let curryJoin = FuncConvert.FuncFromTupled join
[ 1..10 ]
|> List.map string
|> curryJoin "*"				// "1*2*3*4*5*6*7*8*9*10"
let joinStar = curryJoin "*"	// joinStar类型为：string list -> string
```

以上代码将`System.String.Join`转化为F#中的函数，因为该方法具有多个重载，所以第一行代码用来指定一个要转换的重载。

*其实`FuncConvert`类也可以在C#中使用，需要添加`FSharp.Core`程序集，有兴趣的可以自己尝试。*

### C#调用F#代码

本节涉及操作需要创建两个项目，一个**F#的类库项目**，一个**C#的控制台项目**。然后C#项目引用F#项目，因为**涉及到F#中独有类型，还需要引用`FSharp.Core`程序集。**

若要在UWP项目中引用F#项目，需要通过“可移植库”模板创建项目。

因为C#中的类型比F#少了很多，所以很多C#不支持的类型均使用**类**来代替，使用时只需像使用类一样使用它就行了。而模块，在C#中则为**静态类**。

#### F#中的函数

需要注意的是，若在F#将函数作为参数或返回值，则F#中的函数在C#中将会变成

`FSharpFunc<_,_>`对象（位于FSharp.Core程序集的`Microsoft.FSharp.Core`命名空间）。

```F#
//F# 代码，位于TestModule模块
open System
type MathUtilities =
	static member GetAdder() =
		(fun x y z -> Int32.Parse(x) + Int32.Parse(y) + Int32.Parse(z))
```

`GetAdder`函数返回一个将三个字符串转成int再相加的函数，在C#中调用此函数：

```c#
FSharpFunc<string, FSharpFunc<string, FSharpFunc<string, int>>> ss = MathUtilities.GetAdder();
var ret = ss.Invoke("123").Invoke("45").Invoke("67");
```

F#中的`string -> string -> string -> int`类型函数在C#中变成了`FSharpFunc <string, FSharpFunc <string, FSharpFunc <string, int>>>`。

这是因为C#中的**不支持函数柯里化**，如果F#中的函数需要更多的参数，在C#中调用就很麻烦了。虽然在F#使用很方便，但若需要编写供C#使用的程序集，尽量不要使用这些功能。

#### 命名规范

通过上面的了解，至少可以简单地使用F#和C#互相调用。但有个地方可能使有强迫症的程序员很难受：F#模块中的函数命名使用的是**驼峰式（camelCase）**，在C#中类的方法则使用**PascalCase**命名规范。

F#模块在编译成静态类后，在C#中使用变得不一致。在F#中提供了**`CompiledName`**特性用来**指定编译后的名称**。

在第一篇中提到的F#中可用“\`\` \`\`”来使任何字符串作为变量（值）的名称，若想在C#中调用这类值（不符合变量命名规则），也需要用**`CompiledName`**指定编译后的名称，否则无法调用。

```F#
module TestModule
[<CompiledName("Add")>]
let add = fun a b -> a+b
[<CompiledName("IsSeven")>]
let ``7?`` i = i % 7 = 0
```

在C#中调用：

```C#
int i = TestModule.Add(3,4);
var b = TestModule.IsSeven(7);
```





*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 转载请注明源链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-5.html>。*

