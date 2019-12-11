# [翻译]深入C#9



前两天一觉醒来再次收到一线的Bassam Alugili的信,他整理了第二篇文章介绍C#9的最新特性,希望我能把它翻译成中文,让更多中国开发者知道这些新消息.

鄙人不才,英语稍欠火候,这里我对原篇简单翻译,如果感觉都得拗口的可以移步以下链接阅读原文.

https://www.c-sharpcorner.com/article/deev-dive-into-c-sharp-9/

下文提及的这些C＃9功能可以使C＃功能更强大且更实用,阅读此篇文章可以帮助我们为那些必定到来的挑战做好准备,让我们开始吧!  

![](https://disk.iblogs.site/pic/DeepDiveIntoC#9/csharp9Logoversion2.jfif)



官方计划C# 9 是C#的下一个release版本,你可以点击这个[链接](https://github.com/dotnet/csharplang/projects/4#column-4899858)查看C＃语言版本规划.  

![](https://disk.iblogs.site/pic/DeepDiveIntoC#9/LanguageVersion Planning.jfif)

如上图所示,在这个列表中一共有34个计划在C# 9中发布的新功能,当然这只是计划.  

哪些功能会最终发布，发布在那个版本？恐怕只有.Net开发团队才能回答这些问题,他们拥有最终决定权，并且可以随时更改这些提议的功能以及这些功能的语法,所以这篇文章只对现有计划计划进行分析,具体还是以最终发布为准.  



为C＃9计划的最重要的功能是记录类型，可能有区别的并集，更多的模式匹配增强以及现有结构（例如三元和空合并表达式）的其他目标类型。

 

在本文中，我将介绍记录联盟和歧视联盟，并简要介绍其他提案。

 

**!!!!! 重要!!!!!**

 

特别是对于唱片联盟和歧视联盟，它们还没有进入最后阶段。他们俩仍然需要大量工作才能从当前的草人建议语法过渡到最终设计。

 

**记录**

 

我已经等了很久了。记录是轻量型的。它们是名义上的类型，它们可能具有（方法，属性，运算符等），并允许您比较结构相等性！此外，记录属性默认情况下为只读。记录可以是值类型或引用类型。

 

在GitHub上的建议 [在这里](https://github.com/dotnet/csharplang/blob/master/proposals/records.md)。

 

[这里](https://github.com/dotnet/csharplang/blob/master/proposals/recordsv2.md)是最新的记录建议 。

 

*新提案的记录*

 

记录将定义如下，



```c#
public class Point3D  
{  
  public int X { get; set; }  
  public int Y { get; set; }  
  public int Z { get; set; }  
}  
```



防伪类型

 

提出的解决方案是一个新的修饰符*initonly*，可以将其应用于属性和字段，

``` c#
public class Point3D  
{  
  public initonly int X { get; }  
  public initonly int Y { get; }  
  public initonly int Z { get; }  
  ...  
...  
}
```



创建记录

``` c#
void DoSomething()  
{  
  var point3D = new Point3D()  
  {  
    X = 1,  
    Y = 1,  
    Z =1  
  };  
}  
```



***记录旧提案\***

 

示例，下面的记录带有主构造函数

``` c#
data class Point3D(int X, int Y, int Z);
```



相当于

``` c#
public class Demo  
{  
  public void CreatePoint()  
  {  
    var p = new Point3D(1.0, 1.0, 1.0);  
  }  
} 
```



相当于

``` c#
data class Point3D  
{  
  public int X { get; }  
  public int Y { get; }  
  public int Z { get; }  
  public Point(int x, int y, int z)  
  {  
    X = x;  
    Y = y;  
    Z = z;  
  }  
  
  public void Deconstruct(out int X, out int Y, out int Z)  
  {  
    X = this.X;  
    Y = this.Y;  
    Z = this.Z;  
  }  
}  
```



以上的最后一代将是

```c#
class Point3D  
{  
  public initonly int X { get; }  
  public initonly int Y { get; }  
  public initonly int Y { get; }  
  public Point3D(int x, int y, int z)  
  {  
    X = x;  
    Y = y;  
    Y = z;  
  }  
  
  protected Point3D(Point3D other)  
  : this(other.X, other.Y, other.Z)  
  { }  
  
  [WithConstructor]  
  public virtual Point With() => new Point(this);  
  
  public void Deconstruct(out int X, out int Y, out int Z)  
  {  
  X = this.X;  
  Y = this.Y;  
  Z = this.Z;  
  }  
  // Generated equality  
}  
```



## 使用记录和with表达式

 

记录提议是通过新提议的功能“ with-expression”引入的。在编程中，不可变对象是创建后状态无法更改的对象。如果要更改对象，则必须将其复制。“ with”可以帮助您解决问题，并且可以按照以下说明一起使用它们。

```c#
public class Demo  
{  
  public void DoIt()  
  {  
    var point3D = new Point3D() { X = 1, Y = 1, Z =1  };  
    Console.WriteLine(point3D);  
  }  
}  
```



```c#
var newPoint3D = point3D with {X = 42};  
```



创建的新点（newPoint3D）与现有点（point3D）相同，但是X的值更改为42。

 

该建议在模式匹配方面效果很好。

 

## F＃中的记录

 

从F＃MSDN示例复制，输入Point3D = {X：float; Y：浮动；Z：浮动}

```F#
let evaluatePoint (point: Point3D) =  
  match point with  
        | { X = 0.0; Y = 0.0; Z = 0.0 } -> printfn "Point is at the origin."  
        | { X = xVal; Y = 0.0; Z = 0.0 } -> printfn "Point is on the x-axis.  Value is %f." xVal  
        | { X = 0.0; Y = yVal; Z = 0.0 } -> printfn "Point is on the y-axis. Value is %f." yVal  
        | { X = 0.0; Y = 0.0; Z = zVal } -> printfn "Point is on the z-axis. Value is %f." zVal  
        | { X = xVal; Y = yVal; Z = zVal } -> printfn "Point is at (%f, %f, %f)." xVal yVal zVal  
  
evaluatePoint { X = 0.0; Y = 0.0; Z = 0.0 }  
evaluatePoint { X = 100.0; Y = 0.0; Z = 0.0 }  
evaluatePoint { X = 10.0; Y = 0.0; Z = -1.0 }  
```



此代码的输出如下。

 

*Point is at the origin.*

*Point is on the x-axis. Value is 100.000000.*

*Point is at (10.000000, 0.000000, -1.000000).*

 

记录类型由编译器实现，这意味着您必须满足所有这些条件，并且不能弄错它们。

 

因此，它们不仅节省了很多样板，而且消除了整类潜在的错误。

 

而且，此功能在F＃中已经存在了十多年，并且其他语言（例如，Scala，Kotlin）也具有类似的概念。

 

同时支持构造函数和记录的其他语言的示例。

 

F＃



``` c#
type Greeter(name: string) = member this.SayHi() = printfn "Hi, %s" name 
```





斯卡拉

```c#
class Greeter(name: String)  
{  
  def SayHi() = println("Hi, " + name)  
}  
```



## 平等

 

记录按结构而不是参考进行比较

 

**例**

``` c#
void DoSomething()  
{  
    var point3D1 = new Point3D()   
    {  
        X = 1,  
        Y = 1,  
        Z =1  
    };  
  
    var point3D2= new Point3D()   
    {  
        X = 1,  
        Y = 1,  
        Z =1  
    };  
  
    var compareRecords = point3D1 == point3D2; // true  
}  
```



## 歧视联盟

 

区分工会（disjoint union）是从数学中借用的。一个简单的例子来理解这个词。

 

考虑相关的集合，

   **A0 = {（5,0），（6,1）}**

   **A1 = {（7,2）}**

 

然后，可以按以下公式计算出所区分的联合：

 

   **A 0⊔A 1 = {（5,0），（6,1），（7,2）}**

 

正如您在上面看到的那样，所区分的并集是关联集的总和。区分联合（disjoint union）也广泛用于编程语言中（尤其是在FP中），该语言用于对现有数据类型求和。

 

## C＃9中的歧视工会

 

它提供了一种定义可以容纳许多不同数据类型的类型的方法。它们的功能类似于F＃区分的联合。

 

[这里](https://github.com/dotnet/csharplang/blob/master/proposals/discriminated-unions.md)的官方建议 。

 

区分联合对于异构数据很有用；可以包含个别案例，有效案例和错误案例的数据；从一个实例到另一个实例的类型不同的数据。此外，它为小对象层次结构提供了另一种选择。

 

F＃区分联合示例。

```F#
type Person = {firstname:string; lastname:string}  // define a record type   
type ByteOrBool = Y of byte | B of bool  
  
type MixedType =   
    | P of Person        // use the record type defined above  
    | U of ByteOrBool    // use the union type defined above  
  
let unionRecord = MixedType.P({firstname="Bassam"; lastname= "Alugili"});  
let unionType1 =  MixedType.U( B true);   // Boolean type  
let unionType2 =  MixedType.U( Y 86uy);   // Byte type  
```



C＃9歧视联盟示例。

 

使用C＃记录，可以使用联合定义语法的形式将其用C＃表示，例如，

``` c#
// Define a record type  
public class Person  
{  
  public initonly string Firstname { get; }  
  public initonly string Lastname { get; }  
};  
  
enum class ByteOrBool { byte Y; bool B;} // Just for demo the syntax is not fix now.  
```



我们需要的时间是一个表示所有可能的整数加上所有可能的布尔值的类型。

 

![](https://disk.iblogs.site/pic/DeepDiveIntoC#9/cshar9Boolorchar.jfif)

 

换句话说，ByteOrBool是求和类型。在我们的例子中，新类型是字节类型加布尔类型的“和”。与F＃中一样，总和类型称为“区分联合”类型。

``` c#
enum class MixedType  
{  
  Person P;  
  ByteOrBool U;  
}  
```



构造一个联合实例，

``` c#
// Note: the “new” might be not needed!  
var person = new Person()  
{  
  Firstname = ”Bassam”;  
  Lastname = “Alugili”;  
};  
  
var unionRecord = new MixedType.P(person); // Record C# 9  
var unionType1 = new MixedType.U( B true); // Boolean type  
var unionType2 = new MixedType.U( Y 86uy); // Byte type  
```



***歧视工会的使用\***

 

带模式匹配

 

直接使用“子类型”名称和即将出现的匹配表达式。以下是示例，仅供演示人员更好地理解该建议。

 

像Java中一样的异常处理

``` c#
try  
{  
  …  
  …  
}  
catch (CommunicationException | SystemException ex)  
{  
    // Handle the CommunicationException and SystemException here  
}  
```



作为类型约束

``` c#
public class GenericClass<T> where T : T1 | T2 | T3  
```



通用类可以是*T1或T2或T3*类型之一

 

类型化的异构集合

``` c#
var crazyCollectionFP = new List<int|double|string>{1, 2.3, "bassam"};
```



提案中的示例

 

通过不同类型的变量/值/表达式组合的结果类型？：，?? 或切换表达式组合器。

``` c#
var result = x switch { true => "Successful", false => 0 };  
```



 这里的结果类型为-string | int

 

如果某个方法的多个重载具有相同的实现，则Union类型可以完成此工作，

``` c#
void logInput(int input) => Console.WriteLine($"The input is {input}");  
  
void logInput(long input) => Console.WriteLine($"The input is {input}");  
  
void logInput(float input) => Console.WriteLine($"The input is {input}");  
```



可以更改为 



``` c#
void logInput(int|long|float input) => Console.WriteLine($"The input is {input}");
```





 

也许作为返回类型，

``` c#
public int|Exception Method() // returning exception instead of throwing  
  
public class None {}  
  
public typealias Option<T> = T | None; // Option type  
  
public typealias Result<T> = T | Exception; // Result type  
```



更多示例 [在这里](https://github.com/dotnet/csharplang/blob/master/meetings/2018/LDM-2018-05-14.md#records-and-discriminated-unions)。

 

**增强通用类型规范**

 

建议 [在这里](https://github.com/dotnet/csharplang/issues/2823)。

 

**目标类型的空合并（`??`）表达式**

 

这是关于允许从空合并表达式进行隐式转换。

 

**例**

``` c#
void M(List<int> list, uint? u)  
{  
  IEnumerable<int> x = list ?? (IEnumerable<int>)new[] { 1, 2 }; // C# 8  
  var l = u ?? -1u; // C# 8  
}  
  
void M(List<int> list, uint? u)  
{  
  IEnumerable<int> x = list ?? new[] { 1, 2 }; // C# 9  
  
  var l = u ?? -1; // C# 9  
}  
```



**目标类型的隐式数组创建表达式**

 

介绍*“ new（）”*表达式。

 

官方提案示例

``` c#
IEnumerable<KeyValuePair<string, string>> Headers = new[]  
{  
  new KeyValuePair<string, string>("Foo", foo),  
  new KeyValuePair<string, string>("Bar", bar),  
}  
```



可以简化为

``` c#
IEnumerable<KeyValuePair<string, string>> Headers = new KeyValuePair<string, string>[]  
{  
  new("Foo", foo),  
  new("Bar", bar),  
}  
```



但是您仍然需要重复字段/属性初始化程序之后的类型。您可以获得的最接近的是：

``` c#
IEnumerable<KeyValuePair<string, string>> Headers = new[]  
{  
  new KeyValuePair<string, string>("Foo", foo),  
  new("Bar", bar),  
}  
```



为了完整起见，我建议也将new []作为目标类型的表达式。

``` c#
IEnumerable<KeyValuePair<string, string>> Headers = new[]  
{  
  new("Foo", foo),  
  new("Bar", bar),  
}  
```



**目标类型的新表达式**

 

*“ var”*推断左侧，此功能使我们可以推断右侧。

 

**例**

``` c#
Point p = new (x, y);  
ConcurrentDictionary> x = new();  
Mads example: Point[] ps = { new (1, 4), new (3,-2), new (9, 5) }; // all Points 
```



**呼叫者表达属性**

 

允许调用者“字符串化”在呼叫站点传递的表达式。该属性的构造函数将使用一个字符串参数，该参数指定要字符串化的参数的名称。

 

**例** 

``` c#
public static class Verify {    
    public static void InRange(int argument, int low, int high,    
        [CallerArgumentExpression("argument")] string argumentExpression = null,    
        [CallerArgumentExpression("low")] string lowExpression = null,    
        [CallerArgumentExpression("high")] string highExpression = null) {    
        if (argument < low) {    
            throw new ArgumentOutOfRangeException(paramName: argumentExpression, message: $ " {argumentExpression} ({argument}) cannot be less than {lowExpression} ({low}).");    
        }    
        if (argument > high) {    
            throw new ArgumentOutOfRangeException(paramName: argumentExpression, message: $ "{argumentExpression} ({argument}) cannot be greater than {highExpression} ({high}).");    
        }    
    }    
    public static void NotNull < T > (T argument,    
        [CallerArgumentExpression("argument")] string argumentExpression = null)    
    where T: class {    
        if (argument == null) throw new ArgumentNullException(paramName: argumentExpression);    
    }    
}    
  
// CallerArgumentExpression: convert the expressions to a string!      
Verify.NotNull(array); // paramName: "array"      
  
// paramName: "index"      
// Error message by wrong Index:       
"index (-1) cannot be less than 0 (0).", or    
  
// "index (6) cannot be greater than array.Length - 1 (5)."      
Verify.InRange(index, 0, array.Length - 1);    
```



**解构默认**

 

允许以下语法*（int i，字符串s）=默认值；**和**（i，s）=默认值;*

 

**例**

``` c#
(int x, string y) = (default, default); // C# 7  
(int x, string y) = default; // C# 9   
```



**放宽参考和部分修饰符的顺序**

 

在类定义中的ref之前允许使用局部关键字。

 

**例**

``` c#
public ref partial struct {} // C# 7  
public partial ref struct {} // C# 9  
```



**参数空检查**

 

通过在参数上使用小的注释，可以简化对参数的标准null验证。此功能属于代码增强。

 

上次会议记录 [在这里](https://github.com/dotnet/csharplang/blob/master/meetings/2019/LDM-2019-07-10.md#param)。

 

**例**

``` c#
// Before C# 1..7.x  
void DoSomething(string txt)  
{  
    if (txt is null)  
    {  
       throw new ArgumentNullException(nameof(txt));  
     }  
  …  
}  
  
// Candidate for C# 9  
void DoSomething (string txt!)  
{  
  …  
}  
```



**跳过本地人初始化**

 

允许指定System.Runtime.CompilerServices.SkipLocalsInitAttribute作为告诉编译器不发出localsinit标志的方法。SkipLocalsInitiAttribute已添加到CoreCLR。

 

这样的最终结果将是JIT可能不会对本地变量进行零初始化，这在大多数情况下在C＃中是不可观察的。

 

除此以外，stackalloc数据不会被初始化为零。这是可以观察到的，但也是最有启发性的。

 

**Lambda丢弃参数**

 

允许lambda具有名为_的参数的多个声明。在这种情况下，参数是“ discards”并且在lambda内部不可用。

 

**例子**

``` c#
Func zero = (_,_) => 0;  
(_,_) => 1, (int, string) => 1, void local(int , int);  
```



**局部函数的属性**

 

这个想法是允许属性成为局部函数声明的一部分。

 

“根据今天（2019年4月29日）在LDM中的讨论，这将有助于使用[EnumeratorCancellation]的异步迭代器本地函数。

 

我们还应该测试其他属性：”

 

*[DoesNotReturn]*

*[DoesNotReturnIf(bool)]*

*[Disallow/Allow/Maybe/NotNull]*

*[Maybe/NotNullWhen(bool)]*

*[Obsolete]*

 

基本示例

``` c#
static void Main(string[] args)  
{  
  static bool LocalFunc([NotNull] data)  
  {  
    return true;  
  }  
} 
```



此功能的主要用例，

 

将其与实现异步迭代器的本地函数的CancellationToken参数上的EnumeratorCancellation一起使用的另一个示例，这在实现查询运算符时很常见。 

``` c#
public static IAsyncEnumerable Where(this IAsyncEnumerable source, Func predicate)  
{  
  if (source == null)  
      throw new ArgumentNullException(nameof(source));  
    
  if (predicate == null)  
      throw new ArgumentNullException(nameof(predicate));  
  
  return Core();  
  
  async IAsyncEnumerable<T> Core([EnumeratorCancellation] CancellationToken token = default)  
  {  
       await foreach (var item in source.WithCancellation(token))  
       {  
        if (predicate(item))  
        {  
            yield return item;  
        }  
       }  
  }  
}  
```



高级示例 [在这里](https://gist.github.com/rynowak/4d4738a57fb482952056ca67573f1d50)。

 

**本机整数**

 

为本机引入了一组新的本机类型（nint，nuint）。新数据类型的设计计划允许一个C＃源文件使用32位自然或64位存储，具体取决于主机平台类型和编译设置。

 

**例**

 

本机类型取决于操作系统，

```c#
nint nativeInt = 55; take 4 bytes when I compile in 32 Bit host.  
nint nativeInt = 55; take 8 bytes when I compile in 64 Bit host with x64 compilation settings.
```



**函数指针**

 

我记得C / C ++中的函数指针一词。FP是存储函数地址的变量，以后可以通过该函数指针调用该函数的地址。就像在正常的函数调用中一样，可以调用和传递函数指针。

 

建议 [在这里](https://github.com/dotnet/csharplang/blob/master/proposals/function-pointers.md)。

 

新的C＃候选功能之一称为功能指针。C＃函数指针允许使用func *语法声明函数指针。它类似于委托声明使用的语法。

 

**例**

``` c#
unsafe class Example  
{  
  void Example(Action<int> a, delegate*<int, void> f)  
  {  
    a(42);  
    f(42);  
  }  
}  
```



## 总结

 

您已经阅读了有关C＃9功能状态中的候选人的信息，我已向您展示了他们。

 

C＃NEXT功能列表状态如下所示，其中包含C＃9的工作列表。仅当“ master”分支中的候选功能时，这意味着该功能将在[下一版本中](https://github.com/dotnet/roslyn/blob/master/docs/Language Feature Status.md)发布。

 

导入：很多事情仍在讨论中。建议的功能和语法/语义可能会更改，或者功能本身可能会更改或删除。只有.NET开发人员才能决定哪些功能将在C＃9中发布以及何时发布。在下一篇文章中，我将继续提出建议。

 

当C＃9发布时，我将为您制作一份C＃7和C＃8中的备忘单。您可以在GitHub或我的主页上关注我。

- [C＃8](https://github.com/alugili/CSharp8CheatSheet)
- [C＃7.x](https://github.com/alugili/CSharp7Features)