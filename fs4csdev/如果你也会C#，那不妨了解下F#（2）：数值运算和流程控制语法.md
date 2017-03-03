# 数值运算和流程控制语法

## 一些废话
一门语言火不火，与语言本身并没太大关系，主要看语言的推广。
推广得好，用的人多，问题也能及时得到解决，用的人就越多，这是一个良性循环，即使语言本身有很多不足也很快能得到解决。  
但有的语言本身很好，使用者却不多，缺少交流和推广，致使进入恶性循环。
[《黑客与画家》](https://book.douban.com/subject/6021440/)作者把Lisp吹上天，但却没见他继续推广，至今在使用的团队和行业还是很有限。  
而说到F#，国内也出过F#的高校教材，不知道是否有高校开课，在企业上更是很少使用。
“赵姐夫”（[博客](http://blog.zhaojie.me/)）在10年说过要做F#在国内的推广者，几年过去了，也是无声无息。  

那为什么在2016年的现在，那么多新的语言和技术，我们还要来了解F#呢？  

- F#和C#一样，也是基于.Net平台的语言，了解了语法后，就能快速地使用.Net框架甚至C#编写的框架，
  而且在学习过程中.Net框架中很多以前不理解的东西，通过F#就变得很容易理解了。作为.Net程序员，还是值得了解的。
- 函数式语言的“天然支持异步和并行”的能力，也使得多线程开发变得简单。
  C#在最近的版本中经常得益于F#对.Net框架的推进，如加入了`async`关键字，有 `Tuple`了（虽然在语法层面不支持）等等。
- 在最近发布的.Net Core中，也可以通过`dotnet new -l f#`来创建F#项目。*.Net Core里F#的坑，这里就不细说了。* 😅 通过.Net博客，也可以发现越来越多人在使用F#。

**F#并无法替代C#，但两者却能起到互补的作用。**

记得[《七周七语言》](https://book.douban.com/subject/10555435/)里说过：
> 每学一门新的语言，思维方式都会发生改变。编程语言亦是如此。

作者在书中说，每学习一门语言都会去找互动教程，但大多数情况下并找不到称心如意的。
> 若想领会一门语言的精髓，它（指互动教程）可就无能为力了。我想要的是那种痛快淋漓、深入探索语言本质的感觉。 

我是没有作者的这种能通过探索语言本质的能力来介绍F#，但希望也能让大家发现F#语言的**动人心弦之处**。
废话说得有点多了，下面继续介绍F#吧。

## 数值运算
<p id="sample"></p>

我们再翻出[上一篇](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-1.html)的小例子：

```F#
let mutable sum = 0 
for i = 0 to 100 do
    if i%2 <> 0 then sum <- sum + i 
printfn "0到100中的奇数的和为%A" sum ;;
```

四则运算和模运算（取余）不必多说，但对于浮点数（`float`和`float32`）类型，还支持`**`（幂）语法。
```F#
2. ** 3.;;  // val it : float = 8.0
```

像`abs`、`sin`等这些**基本数学函数**在F#里也包含在操作符（Operators）的模块里，所有操作符可在[MSDN文章（Core.Operators Module(F#)）](https://msdn.microsoft.com/visualfsharpdocs/conceptual/core.operators-module-%5bfsharp%5d)查看。

### 大整数（BigInteger）
在[上一篇](http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-1.html)的数值类型对比中，我们可以看出，F#在语法层面上支持大整数。  
大整数于.Net 4.0添加到框架中，在`System.Numerics`模块里。F#创建项目时默认引用了此dll。（大整数也支持`**`运算符，但指数必须为`int`，其实就是`Pow`方法。）
```F#f
let bigInt = 4325I;; //相当于C#的 var bigInt = new BigInteger(4325);
3I ** 8;;  // val it : System.Numerics.BigInteger = 6561 ...
```
在内存足够的情况下，`BigInteger`支持任意大的整数。 *通过大整数，我们也可以自己实现大有理数，或直接使用第三方类库。*

### 数值类型转换
在F#中，**没有隐式类型转换**，就连`int`到`long`或`float`也没有。所以，数值类型转换使用返回对应类型的转换函数：
```F#
int 2.5;;   // 2
float "3.1415"  // 3.1415
(float 2)**(float 3);;  //8.0
```
当然也可以使用.Net框架的`System.Convert`静态类。上述方法在转换字符串时也是调用的此方法。

### 位运算符
F#中的位运算符均使用三重符号：

|  运算符   |  名称  | 示例                   |    结果 |
| :----: | :--: | :------------------- | ----: |
|  &&&   |  与   | 0b1111 &&& 0b0011    |     3 |
| \|\|\| |  或   | 0xFF00 \|\|\| 0x00FF | 65535 |
|  ~~~   |  取反  | ~~~0b0011            |    -4 |
|  ^^^   |  异或  | 0b0011 ^^^ 0b0101    |     6 |
|  <<<   |  左移  | 0b0001 <<< 3         |     8 |
|  >>>   |  右移  | 0b1000 >>> 3         |     1 |


### 比较
因为函数式语言中，数值不可变，所以`=`在F#中并不是赋值符号。
**`=`在F#执行相等比较操作，而不等则为`<>`**，这两个符号与C#有异：
```F#
5=8 ;;  //false
"ax"<>"ay" ;;   //true
```
那就会有人说，F#也有可变的值啊，怎么赋值呢！没错，在上面的[示例代码](#sample)中`sum`为可变值，每次循环通过`<-`符号改变其值。
*好诡异但又好有道理的符号啊！* 😲  

**所以需要注意，C#中的 `int i = 1;` 其实是相当于F#中的 `let mutable i = 1` 。因为C#默认创建的是可变类型。** *请查看后面的[while循环](#while)介绍。*

此外，F#中还有一个`compare`函数用来比较两个值：
```F#
compare 31 31;; //  0
compare 5 4;;   //  1
(*
    compare x y
    若x > y则返回1；若x < y则返回-1；若x = y则返回0    
*)
```
这个比较在C#中有实现过`IComparable`接口的应该了解，其实`compare`函数的参数就是需要实现了`IComparable`接口的类型。  
在.Net中，比较是一个复杂的话题，在此就不展开了，可参考MSDN文章（[Object.Equals方法](https://msdn.microsoft.com/zh-cn/library/bsc2ak47(v=vs.110).aspx)和[IComparable 接口](https://msdn.microsoft.com/zh-cn/library/system.icomparable(v=vs.110).aspx)）。

## 流程控制语法
### for循环语句
下面是0到100的循环，从0打印到100：
- C# 代码：
```F#
for (int i=0; i<=100; i++) { System.Console.WriteLine(i); }
```
- F# 代码：
```F#
for i=0 to 100 do printfn "%i" i
for i in [0..100] do printfn "%i" i
for i=100 downto 0 do printfn "%i" i //逆向循环，从100到0
```

F#版本中的`[0..100]`是创建一个从0到100的列表（`list`），在之后列表模块会进行介绍。  
遗憾的是`for ... to`和`for ... downto`语句无法创建如`for (int i=0; i<=100; i+=2)`这种间隔不为1的循环，只能使用`for ... in`再加上一个列表。

### <p id="while">while循环语句</p>
用`while`循环实现的从0打印到100：
- C# 代码：
```F#
int i = 0;
while (i <= 100)
{
    System.Console.WriteLine(i);
    i++;
}
```

- F# 代码：
```F#
let mutable i = 0
while i <= 100 do
    printfn "%i"
    i <- i + 1
```
需要注意的是，F#中并没有`do...while`的循环，在`for`循环和`while`循环中，也都没有`break`和`continue`关键字。  
刚开始接触F#或习惯了C#的循环语句可能会不习惯，但**当你熟悉了函数式编程的思想后，会发现这些在F#中都是不重要的，甚至循环语句都是不必要的**。  
同时，也尽量不要使用可变类型。因为值的不可变性，使多线程编程变得简单；而在C#中，因为值默认为可变的，使程序在多线程运行中增加了很多不确定因素。

### 条件语句之if
在上面的[示例代码](#sample)中已经使用过`if`语句。
在F#中，`if`语句的完整结构是这样的：
```F#
if x>y then "x大于y"
elif x<y then "x小于y"
else "x等于y"

if x>y then printfn "x大于y"  //if分支无返回值
```

在F#的if语句中，每个分支必须返回相同类型的值，但不需要`return`关键字（在介绍函数时会涉及）。  
**如果if分支没有返回值（即返回`unit`），则else为可选的；否则，必须有else分支。**（类似于C#的`?:`操作符，但F#中没有这个操作符。）  
C#中的`else if`在F#中为`elif`。

### 条件语句之switch(match)
在C#中，还有一种分支结构的语句是使用`switch...case`。在F#中类似的语法是`match...with`。
- C# 代码：
```c#
int i = 1;
switch (i)
{
    case 1:
        Console.WriteLine("this is one");
        break;
    case 2:
        Console.WriteLine("this is two");
        break;
    case 3:
        Console.WriteLine("this is three");
        break;
    default:
        Console.WriteLine("this is something else");
        break;
}
```

- F# 代码：
```F#
let i = 1
match i with
    | 1 -> printfn "this is one"
    | 2 -> printfn "this is two"
    | 3 -> printfn "this is three"
    | _ -> printfn "this is anything else"
```

其中`_`为通配符，与C#`switch`中的`default`类似。  
但F#中的`match`语句功能非常强大，可查看MSDN文章[Match表达式（Match Expressions）](https://msdn.microsoft.com/visualfsharpdocs/conceptual/match-expressions-%5bfsharp%5d)和
[模式匹配（Pattern Matching）](https://msdn.microsoft.com/visualfsharpdocs/conceptual/pattern-matching-%5bfsharp%5d)。
**模式匹配**在函数式编程中有着作用，在之后的函数式编程中也会进行介绍。

*在下一篇中，我们来了解数组和列表，以及一些F#特有的类型。*

*本文发表于[博客园](http://www.cnblogs.com/hjklin)。 转载请注明源链接：<http://www.cnblogs.com/hjklin/p/fs-for-cs-dev-2.html>。*

