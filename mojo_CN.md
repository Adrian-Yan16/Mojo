## Mojo🔥编程手册

一个全面的 Mojo 语言特性及其代码示例之旅。


Mojo是一种编程语言，它像Python一样易于使用，但具有C++和Rust的性能。此外，Mojo还提供了利用整个Python库生态系统的能力。


Mojo通过利用集成缓存、多线程和云分布技术的下一代编译器技术来实现这一壮举。此外，Mojo的自动调整和编译时元编程功能可以让您编写可移植到最奇特硬件的代码。


更重要的是，Mojo让你可以利用整个Python生态系统，这样你就可以继续使用你熟悉的工具。Mojo旨在通过保留Python的动态特性，同时添加新的[系统编程](https://en.wikipedia.org/wiki/Systems_programming)和[并发.这些新的系统编程原语将允许Mojo开发人员构建当前需要C、C++、Rust、CUDA和其他加速器系统的高性能库。通过将动态语言和系统语言的优点结合起来，我们希望提供一种**统一**的编程模型，它能够适用于不同的抽象层次，对初学者友好，并可以在多种用例中进行扩展，包括加速器、应用程序编程和脚本编写。


这份文档是对Mojo编程语言的介绍，而不是一份完整的语言指南。它假定了对Python和系统编程概念的了解。目前，Mojo仍处于开发阶段，文档面向具有系统编程经验的开发人员。随着语言的发展和更广泛的可用性，我们希望它对每个人，包括初学者程序员，都友好和易于访问。今天它就是不在那里。

## 使用Mojo编译器


你可以像运行Python程序一样，从终端运行Mojo程序。如果你有一个叫做`hello.mojo`（或者`hello.🔥`——是的，文件扩展名可以是表情符号！）的文件，只需要输入`mojo hello.mojo`:

```auto
$ cat hello.🔥
def main():
    print("hello world")
    for x in range(9, 0, -3):
        print(x)
$ mojo hello.🔥
hello world
9
6
3
$
```

再次，您可以使用表情符号或 `.mojo` 后缀。

如果你有兴趣深入了解Mojo的内部实现细节，可以参考标准库中的类型、笔记本、博客和其他示例代码

## 基本系统编程扩展


鉴于我们的兼容性目标和Python在高级应用和动态API方面的优势，我们不必花太多时间来解释语言的这些部分。另一方面，Python对系统编程的支持主要委托给C，我们希望提供一个在这个世界中也很棒的单一系统。因此，本节将对每个主要组件和功能进行详细分解，并通过示例说明如何使用它们。

## `let` 和 `var` 声明


在Mojo中的`def`内部，你可以给一个名字赋值，它会隐式地创建一个函数作用域变量，就像在Python中一样。这提供了一种非常动态且低仪式的编写代码方式，但它也存在两个挑战：

1. 系统程序员经常希望声明一个值是不可变的，以保证类型安全和性能。
1. 如果他们在赋值过程中错误地输入变量名，他们可能会得到一个错误。


为了支持这一点，Mojo 提供了作用域运行时值声明：`let` 是不可变的，而 `var` 是可变的。这些值使用词法作用域并支持名称阴影。

```auto
def your_function(a, b):
    let c = a
    c = b  # error: c is immutable

    if c != b:
        var c = b
        stuff()
```

`let`和`var`声明支持类型说明符以及模式，以及后期初始化。

```auto
def your_function():
    let x: SI8 = 42
    let y: SI64 = 17

    let z: SI8
    if x != 0:
        z = 1
    else:
        z = foo()
    use(z)
```


注意，在`def`声明中，`let`和`var`完全是可选的。您仍然可以像使用Python一样使用隐式声明的值，它们会按照惯例具有函数作用域。

## `struct`类型


Mojo基于MLIR和LLVM，它们提供了一个尖端的编译器和代码生成系统，被用于许多编程语言。这让我们更好地控制数据组织，直接访问数据字段以及其他改善性能的方式。现代系统编程语言的一个重要特性是，可以在复杂的低级操作之上构建高级和安全的抽象，而不会导致在Mojo中，这由`struct`类型提供。


Mojo中的`struct`类似于Python的`class`：它们都支持方法、字段、运算符重载、用于元编程的装饰器等。他们的差异如下：

+ Python类是动态的：它们允许动态分派、猴子补丁（或“混淆”）以及在运行时动态绑定实例属性。

+ Mojo结构是静态的：它们在编译时绑定（您不能在运行时添加方法）结构允许您在安全且易于使用的同时，以灵活性换取性能。


这是一个结构的简单定义：

```auto
@value
struct MyPair:
    var first: Int
    var second: Int
    def __lt__(self, rhs: MyPair) -> Bool:
        return self.first < rhs.first or
              (self.first == rhs.first and
               self.second < rhs.second)
```

语法上，与Python `class`相比，`struct`中所有实例属性**必须**使用`var`或`let`声明明确声明。


在Mojo中，“struct”的结构和内容是预先设定的，在程序运行期间不能更改。与Python不同，Mojo不允许对结构体进行即时的增加、删除或更改属性。这意味着你不能使用`del`在程序运行中移除一个方法或改变它的值。

然而，`struct`的静态特性有一些很大的好处！它有助于Mojo更快、更可靠地运行您的代码。程序精确地知道在哪里找到结构信息，以及如何在没有任何额外步骤或延迟的情况下使用它。

Mojo的结构也非常适合您可能已经熟悉的Python特性，例如运算符重载（可以让您更改`+`和`-`等数。此外，所有的“标准类型”（如`Int`、`Bool`、`String`甚至`Tuple`）都是使用结构体制作的。这意味着它们是标准工具集的一部分，而不是内置于语言本身中。这给你在编写代码时更多的灵活性和控制权。


如果你想知道`inout`在`self`参数上意味着什么：这表明该参数是可变的，函数内部所做的更改对调用者可见。有关[inout参数](#inout-arguments)的详细信息，请参见下面。

## `Int` 是一个类型，而 `int` 是一个基本数据类型。

在Mojo中，你可能会注意到我们使用`Int`（大写“I”），这与Python的`int`（小写“i”）不同。这种差异是有目的的，而且实际上是件好事！


在Python中，`int`类型可以处理非常大的数字，并具有一些额外的功能，比如检查两个数字是否为同一个对象。但这伴随着一些额外的负担，可能会减慢事情的进度。Mojo的`Int`则不同。它被设计为简单、快速，并且针对您的计算机硬件进行了优化，以便快速处理。

我们做出这个选择的两个主要原因是：

1. 我们希望为需要与计算机硬件紧密合作的程序员（系统程序员）提供一种透明可靠的与硬件交互的我们不想依赖于花哨的技巧（比如JIT编译器）来使事情变得更快。

1. 我们希望Mojo能够与Python无缝配合，不会出现任何问题。通过使用不同的名称（Int而不是int），我们可以在不改变Python的int工作方式的情况下将两种类型都保留在Mojo中。

作为额外奖励，`Int` 遵循与 Mojo 中你可能创建的其他自定义数据类型相同的命名风格。此外，`Int`是Mojo标准工具集中包含的`struct`。

## 强类型检查


尽管你仍然可以像在Python中一样使用灵活的类型，但Mojo让你使用严格的类型检查。类型检查可以使您的代码更加可预测、可管理和安全。


Mojo的`struct`类型是采用强类型检查的主要方法之一。Mojo中的`struct`定义定义了一个编译时绑定的名称，并且在类型上下文中对该名称的引用被视为对正在定义的值的例如，考虑使用上面显示的`MyPair`结构的以下代码：

```auto
def pairTest() -> Bool:
    let p = MyPair(1, 2)
    return p < 4 # gives a compile-time error
```

当你运行这段代码时，你会得到一个编译时错误，告诉你“4”无法转换为`MyPair`，而这是`MyPair.__lt__`右侧所需的


这是使用系统编程语言工作时的一种常见经历，但这不是Python的工作方式。Python具有用于[MyPy](https://mypy.readthedocs.io/)类型注释的语法上完全相同的特性，但它们不会被编译器通过将类型与特定声明相关联，Mojo可以处理经典的类型注释提示和强类型规范，而不破坏兼容性

强类型不仅仅用于类型检查。由于我们知道类型是准确的，因此我们可以根据这些类型优化代码，将值传递到寄存器中，并且在参这是Mojo为系统程序员提供的安全和可预测性保证的基础。

## Overloaded函数和方法


像Python一样，在Mojo中可以定义函数而无需指定参数数据类型，Mojo将自动处理它们。当你想要表达式的 API，只需接受任意输入，让动态分派决定如何处理数据时，这是很不错的。然而，当您想要确保类型安全时，正如上面所讨论的，Mojo也提供了对重载函数和方法的完整支持。


这允许您使用相同的名称定义多个具有不同参数的函数。这是许多语言（如C++、Java和Swift）中常见的特性。


在解析函数调用时，Mojo会尝试每个候选者，如果只有一个可以工作，它会使用该候选者；如果可以确.在后一种情况下，您可以通过在调用站点上添加显式转换来解决歧义。让我们来看一个例子：

```auto
struct Array[T: AnyType]:
    fn __getitem__(self, idx: Int) -> T: ...
    fn __getitem__(self, idx: Range) -> ArraySlice: ...
```

你可以在结构体和类中重载方法，以及重载模块级函数。


Mojo不仅仅支持结果类型的重载，而且不使用结果类型或上下文类型信息进行类型推断，从而使事Mojo永远不会产生“表达式太复杂”的错误，因为它的类型检查器在定义上是简单而快速的。


如果你没有给参数定义类型，那么这个函数的行为就和Python中的动态类型一样。一旦定义了单个参数类型，Mojo 就会查找重载候选项并按照上述方式解析函数调用。


函数和方法可以根据参数和参数签名进行重载。有关更多信息以及关于重载解析规则的描述，请参见[参数重载](#overloading-on-parameters)。

## `fn` 定义


上述扩展是提供低级编程和抽象能力的基石，但许多系统程序员比Mojo中的`def`更喜欢更多的控制和可预测性。简而言之，`def`被设计成非常动态、灵活，并且与Python的一般规则兼容：参数是可变的，局部变量在首次使用时隐式声明，且作用域并不受限制。这对于高级编程和脚本编写很棒，但不总是很适合系统编程。为了补充，Mojo提供了一个`fn`声明，它就像`def`的“严格模式”。

另一种方案：与其使用新的关键字`fn`，我们可以在现有的函数定义中添加一个修饰符或装饰器`@strict def`。然而，我们仍然需要采用新的关键字，而且这样做的成本很低。在系统编程领域的实践中，`fn`经常被使用，因此将其纳入第一类可能是明智的。

就调用者而言，`fn` 和 `def` 是可以互换的：没有任何一个 `def` 能提供的 `fn` 无法提供的（反之亦然）。`fn`的不同之处在于，它在其函数体内部更加受限和受控（或者说：更加拘泥和严格）。与`def`函数相比，`fn`有许多限制：

1. 函数体内的参数值默认为不可变（类似于`let`），而不是可变（类似于`var`）。这可以捕获意外的变异，并允许使用不可复制的类型作为参数。
1. 参数值需要指定类型（除了方法中的`self`外），以捕获意外省略类型规范。类似地，如果缺少返回类型说明符，则解释为返回`None`而不是未知的返回类型。注意，两者都可以显式声明返回`object`，这允许人们有选择地采用`def`的行为。
1. 禁止隐式声明局部变量，因此所有局部变量必须明确声明。这捕获名称错误，并与`let`和`var`提供的作用域相协调。
1. 两者都支持抛出异常，但必须在`fn`上使用`raises`关键字明确声明。



编程模式在团队之间会有很大的差异，而这种严格程度不适用于所有人。我们预计习惯于C++的人们会更倾向于使用`fn`，而高级程序员和机器学习研究人员会继续使用`def`。Mojo允许您自由地混合`def`和`fn`声明，例如。实现一些方法用一种，另一些方法用另一种，并允许每个团队或程序员决定什么最适合他们的用例。

有关Mojo函数中参数行为的更多信息，请参见下面关于[参数传递控制和内存所有权](#argument-passing-control-and-memory-ownership)的部分。

## `__copyinit__`和`__moveinit__`特殊方法

Mojo支持像C++和Swift这样的语言中的完整“值语义”，并且它通过使用`@value`装饰器（如下文详细描述），使得定义字段的简单聚合变得非常容易。

对于高级用例，Mojo允许您定义自定义构造函数（使用Python的现有`__init__`特殊方法）、自定义析构函数（使用现有的`__del__`特殊方法）以及使用新的`__copyinit__`和`__moveinit__`特殊方法来定义自定义复制和移动构造函数。

这些低级别的定制钩子在进行低级别的系统编程时很有用，例如在编写设备驱动程序时。手动内存管理。例如，考虑一个需要在构造时为字符串数据分配内存，并在值被销毁时销毁它的动态字符串类

```auto
struct MyString:
    var data: Pointer[UI8]

    # StringRef has a data + length field
    def __init__(inout self, input: StringRef):
        let data = Pointer[UI8].alloc(input.length+1)
        data.memcpy(input.data, input.length)
        data[input.length] = 0
        self.data = Pointer[UI8](data)

    def __del__(owned self):
        self.data.free()
```


这个`MyString`类型使用低级函数实现，以展示这种工作方式的简单示例——一个更现实的实现将使用短字符串。然而，如果你继续尝试，你可能会感到惊讶：

```auto
fn useStrings():
    var a: MyString = "hello"
    print(a)   # Should print "hello"
    var b = a  # ERROR: MyString doesn't implement __copyinit__

    a = "Goodbye"
    print(b)   # Should print "hello"
    print(a)   # Should print "Goodbye"
```

Mojo编译器不允许我们从`a`复制字符串到`b`，因为它不知道如何处理该操作。`MyString`包含一个`Pointer`实例（等价于低级C指针），而Mojo不知道它指向什么数据或如何复制它。更一般地说，某些类型（如原子号）不能复制或移动，因为它们的地址提供了一种身份，就像类实例


在这种情况下，我们确实希望我们的字符串可以复制。为了实现这一点，我们实现了`__copyinit__`特殊方法，通常的实现方式如下：

```auto
struct MyString:
    ...
    def __copyinit__(inout self, existing: Self):
        self.data = Pointer(strdup(existing.data.address))
```


使用这种实现，我们上面的代码可以正确工作，而`b = a`复制产生了一个逻辑上独立的字符串实例，它有着自己的生命周期和数据。上面的代码指示使用C风格的`strdup()`函数进行复制。Mojo同样支持`__moveinit__`方法，它允许Rust风格的移动（在生命周期结束时获取一个值）和C++风格的移动（移有关值生命周期的更多讨论，请参见下面的[值生命周期](#value-lifecycle)部分。


Mojo提供对值的生命周期的完全控制，包括使类型可复制、只能移动和不可移动的能力。这比像Swift和Rust这样的语言提供的控制力更强，它们要求值至少是可移动的。如果你想知道`existing`如何在不创建副本的情况下传入`__copyinit__`方法，请查看下面[借用参数](#borrowed-arguments)一节。

## 参数传递控制和内存所有权

在Python和Mojo中，大部分语言都围绕函数调用：许多（表面上）内置的行为都是使用“双下划线”。在这些魔法函数内部，通过参数传递确定了大量的内存所有权。

让我们复习一下关于Python和Mojo如何传递参数的一些细节：

+ 所有传入*Python* `def`函数的值都使用引用语义。这意味着该函数可以修改传入其中的可变对象，并且这些更改在函数外可见。然而，对于不熟悉的人来说，这种行为有时令人惊讶，因为你可以改变参数指向的对象，而这个改变在函数外部是不可见的。

+ 默认情况下，传入*Mojo* `def`函数的所有值都使用值语义。与Python相比，这是一个重要的区别：Mojo的`def`函数接收所有参数的副本——它可以在函数内部修改参数，但是函数内部对参数的修改在函数外部是不可见的。

+ 所有传入Mojo [`fn`函数](#fn-definitions)的值默认情况下都是不可变的引用。这意味着该函数可以读取原始对象（它不是副本），但它无法对对象进行任何修改。


这种在Mojo `fn`中不可变参数传递的约定被称为“借用”。在接下来的章节中，我们将解释如何更改Mojo中`def`和 `fn`函数

## 为什么参数约定很重要


在Python中，所有基本值都是对象的引用——如上所述，Python函数可以修改原始对象。因此，Python开发人员习惯于将一切都视为引用语义。然而，在CPython或机器级别上，您可以看到引用本身实际上是*按副本*传递的，即Python复制一个指针并调整引用计数。

这种Python方法为大多数人提供了一种舒适的编程模型，但它要求所有值都必须在堆上分配（由于引用共享。Mojo classes（TODO：will）遵循相同的引用语义方法来处理大多数对象，但是在系统编程上下文中，对于简单类型如整型，这种方式并不实用。在这些情况下，我们希望这些值存放在堆栈中或者甚至是硬件寄存器中。因此，Mojo structs总是被内联到它们的容器中，无论是作为另一种类型的字段还是被内联到包含函数的堆栈帧中。

这引发了一些有趣的问题：如何实现需要修改结构类型`self`的方法，比如`__iadd__`？`let`是如何工作的，那它如何防止突变？如何控制这些值的生命周期以保持 Mojo 的内存安全性？


答案是，Mojo编译器使用数据流分析和类型注释，以提供对值复制、引用别名和突变控制的完全控这些特性在许多方面与Rust语言中的特性相似，但它们的工作方式有所不同，以便使Mojo更容易学习，并且它们更好地与Python生态系统集成，而无需承担沉重的注解负担。

在接下来的章节中，您将学习如何控制传入Mojo `fn`函数的对象的内存所有权。

## 不可变参数（借用）


一个借用的对象是一个函数接收到的对象的**不可变引用**，而不是接收到对象的副本。因此，被调用的函数对该对象具有完全的读取和执行访问权限，但它不能修改它（调用者仍然对该对象拥有独占的“所有权”。）

尽管这是`fn`参数的默认行为，但如果您愿意，可以使用`borrowed`关键字明确定义它（您还可以将`borrowed`应用

```auto
fn use_something_big(borrowed a: SomethingBig, b: SomethingBig):
    """'a' and 'b' are passed the same, because 'borrowed' is the default."""
    a.print_id()
    b.print_id()
```


这个默认值适用于所有参数，包括方法的`self`参数。当传递大值或者传递昂贵的值（比如Python/Mojo类默认的引用计数指针）时，这种方式更加高效，因为传以下是基于上面代码的更详细的示例：

```auto
# A type that is so expensive to copy around we don't even have a
# __copyinit__ method.
struct SomethingBig:
    var id_number: Int
    var huge: InlinedArray[Int, 100000]
    fn __init__(inout self): …

    # self is passed inout for mutation as described above.
    fn set_id(inout self, number: Int):
        self.id_number = number

    # Arguments like self are passed as borrowed by default.
    fn print_id(self):  # Same as: fn print_id(borrowed self):
        print(self.id_number)

fn try_something_big():
    # Big thing sits on the stack: after we construct it it cannot be
    # moved or copied.
    let big = SomethingBig()
    # We still want to do useful things with it though!
    big.print_id()
    # Do other things with it.
    use_something_big(big, big)
```


因为`fn`函数的默认参数约定是`借用`，Mojo具有简单而逻辑性的代码，默认情况下做正确的事情。例如，我们不想复制或移动整个`SomethingBig`只是为了调用`print_id()`方法，或者在调用`use_something_big()`时。


这种借用参数约定在某些方面与C++中通过`const&`传递参数类似，这样可以避免值的拷贝，并且在被调但是，借用的约定与C++中的`const&`有两个重要的不同：

1. Mojo编译器实现了一个借用检查器（类似于Rust），该检查器可以防止代码在存在不可变引用的情况下。你可以有多个借用（就像上面调用`use_something_big`一样），但是你不能同时传递可变引用并借用它。目前尚未启用。
1. 小值如`Int`、`Float`和`SIMD`直接通过机器寄存器传递，而不是通过额外的间接引用（这是因为它们使用[`@register_passable`装与C++和Rust等语言相比，这是一个重大的性能增强，将这种优化从每个调用站点移动到声明在类型上。

与Rust类似，Mojo的借用检查器强制执行不变量的排他性。Rust和Mojo的主要区别是，Mojo不需要在调用者端使用符号来传递借用。此外，Mojo在传递小值时更有效率，而Rust默认情况下是移动值而不是通过借用来传递它们。这些政策和语法决定使Mojo能够提供一种更易于使用的编程模型。

## 可变参数（`inout`）

另一方面，如果您定义了一个`fn`函数，并希望一个参数是可变的（为了使函数内部对参数的更改在函数外部可见），您必须使用 `inout` 关键字将参数声明为可变的。

以下是一个例子，其中`__iadd__`函数试图修改`self`：

```auto
struct Int:
    # self and rhs are both immutable in __add__.
    fn __add__(self, rhs: Int) -> Int: ...

    # ... but this cannot work for __iadd__
    fn __iadd__(self, rhs: Int):
        self = self + rhs  # ERROR: cannot assign to self!
```

这里的问题是`self`是不可变的，因为这是一个Mojo `fn`函数，所以它不能改变它被调用时的参数的内部状态。解决方案是通过在`self`参数名称上添加`inout`关键字声明参数是可变的：

```auto
struct Int:
    # ...
    fn __iadd__(inout self, rhs: Int):
        self = self + rhs    # OK
```

**提示：** 当你看到`inout`时，意味着函数内部对参数所做的任何更改都是在函数外可见的。

现在，函数中的`self`参数是可变的，并且调用者中的任何更改都是可见的——即使调用者有一个非常复杂的计算来访问，例如数组下标

```auto
fn show_mutation():
    var x = 42
    x += 1
    print(x)    # prints 43 of course

    var a = InlinedFixedVector[16, Int](...)
    a[4] = 7
    a[4] += 1    # Mutate an element within the InlinedFixedVector
    print(a[4])  # Prints 8

    let y = x
    y += 1       # ERROR: Cannot mutate 'let' value
```


Mojo实现了上面`InlinedFixedVector`元素的原地突变，通过将一个调用`__getitem__`的调用发送到临时缓冲区，然后在调用之后使用 `__setitem__` 进行存储。对 `let` 值进行修改失败，因为无法将不可变值形成可变引用。同样，如果实现了 `__getitem__` 而没有实现 `__setitem__`，则编译器会拒绝尝试使用带有 `inout` 参数的下标操作。

当然，你可以声明多个`inout`参数。例如，您可以定义并使用一个像这样的swap函数：

```auto
fn swap(inout lhs: Int, inout rhs: Int):
    let tmp = lhs
    lhs = rhs
    rhs = tmp

fn show_swap():
    var x = 42
    var y = 12
    swap(x, y)
    print(x)  # Prints 12
    print(y)  # Prints 42
```

这个系统的一个非常重要的方面是它能够正确组合。

注意，我们不把这种参数传递称为“引用传递”。尽管`inout`约定在概念上是相同的，但我们不称之为“按引用传递”，因为实际上实现可能使用指针来传递值。

## 传递参数（`owned` 和 `^`）


Mojo支持的最终参数约定是`owned`参数约定。这个约定用于希望获得值的独占所有权的函数，通常与后缀`^`运算符一起使用。


例如，想象你正在使用一个像唯一指针这样的只能移动的类型。使用借用约定可以轻松地处理独特的指针，但在某些时候，您可能希望将所有权转移到其他函数。这就是 `^` “转移”操作符的作用：

```auto
fn usePointer():
    let ptr = SomeUniquePtr(...)
    use(ptr)        # Perfectly fine to pass to borrowing function.
    take_ptr(ptr^)  # Pass ownership of the `ptr` value to another function.

    use(ptr) # ERROR: ptr is no longer valid here!
```

对于可移动类型，`^`操作符结束值绑定的生命周期，并将值所有权转移到其他地方（在本例中，是`take_ptr()` function)。为了支持这一点，你可以定义函数接受`owned`参数。例如，您可以这样定义`take_ptr()`：

```auto
fn take_ptr(owned p: SomeUniquePtr):
    use(p)
```

因为它被声明为`owned`，`take_ptr()`函数知道它对值有独特的访问权，并可以安全地将指针的所有权这对于像独特指针这样的东西非常重要，当你想避免拷贝时也很有用。


例如，您会在析构函数和消耗移动构造函数上显着看到`owned`约定。例如，我们之前定义的`MyString`类型可以如下定义：

```auto
struct MyString:
    var data: Pointer[UI8]

    # StringRef has a data + length field
    def __init__(inout self, input: StringRef): ...
    def __copyinit__(inout self, existing: Self): ...

    def __moveinit__(inout self, owned existing: Self):
        self.data = existing.data

    def __del__(owned self):
        self.data.free()
```

在`__del__`函数中指定`owned`是很重要的，因为你必须拥有一个值才能销毁它。

## 比较`def`和`fn`参数传递

Mojo的`def`函数本质上只是为`fn`函数提供的语法糖。

+ 一个没有显式类型注释的`def`参数默认为`Object`。

+ 一个没有约定关键字（例如`inout`或`owned`）的`def`参数将通过隐式复制传入具有与参数同名的可变var中。这要求该类型具有一个`__copyinit__`方法。


例如，这两个函数具有相同的行为：

```auto
def example(inout a: Int, b: Int, c):
    # b and c use value semantics so they're mutable in the function
    ...

fn example(inout a: Int, b_in: Int, c_in: Object):
    # b_in and c_in are immutable references, so we make mutable shadow copies
    var b = b_in
    var c = c_in
    ...
```


shadow copies通常不会增加任何开销，因为像`Object`这样的小类型的引用很便宜可以复制。调整引用计数是昂贵的部分，但通过移动优化可以消除这一部分。

## Python集成


使用你熟悉和喜爱的Python模块在Mojo中很容易。你可以将任何Python模块导入到你的Mojo程序中，并从Mojo类型中创建Python类型。

## 导入Python模块

在Mojo中导入Python模块，只需调用`Python.import_module()`并传入模块名称即可：

```auto
from PythonInterface import Python

# This is equivalent to Python's `import numpy as np`
let np = Python.import_module("numpy")

# Now use numpy as if writing in Python
a = np.array([1, 2, 3])
print(a)
```

是的，这导入了Python NumPy，你可以导入*任何其他的Python模块*。

目前，您无法导入单个成员（例如单个Python类或函数）-您必须导入整个Python模块，然后通过模块名称

## 导入本地Python模块

如果您想在Mojo中使用某些本地Python代码，只需将该目录添加到Python路径中，然后导入模块即可。

例如，假设你有一个像这样的Python文件：

```python
import numpy as np

def my_algorithm(a, b):
    array_a = np.random.rand(a, a)
    return array_a + b
```

这是你如何在Mojo中导入和使用它的方法：

```auto
from PythonInterface import Python

Python.add_to_path("path/to/module")
let mypython = Python.import_module("mypython")

let c = mypython.my_algorithm(2, 3)
print(c)
```


使用Python在Mojo中无需担心内存管理。因为Mojo从一开始就是为Python设计的，所以一切都能正常运行。

## Python中的Mojo类型

Mojo原始类型隐式转换为Python对象。今天我们支持列表、元组、整数、浮点数、布尔值和字符串。

例如，给定以下Python函数，用于打印Python类型：

```python
def type_printer(my_list, my_tuple, my_int, my_string, my_float):
    print(type(my_list))
    print(type(my_tuple))
    print(type(my_int))
    print(type(my_string))
    print(type(my_float))
```

您可以导入它并传递Mojo类型，没有问题。

```auto
from PythonInterface import Python

Python.add_to_path("/path/to/module")
let mypython2 = Python.import_module("mypython2")
mypython2.type_printer([0, 3], (False, True), 4, "orange", 3.4)
```

它将在隐式转换为Python类型后输出类型：

```python
<class 'list'>
<class 'tuple'>
<class 'int'>
<class 'str'>
<class 'float'>
```


Mojo还没有标准字典，因此目前还不可能从Mojo字典中创建Python字典。您可以在Mojo中使用Python字典！要创建一个Python字典，请使用`dict`方法：

```auto
from PythonInterface import Python
from PythonObject import PythonObject
from IO import print
from Range import range

def main() -> None:
    let dictionary = Python.dict()
    dictionary["fruit"] = "apple"
    dictionary["starch"] = "potato"
    let keys: PythonObject = ["fruit", "starch", "protein"]
    let N: Int = keys.__len__().to_index()
    print(N, "items")
    for i in range(N):
        if Python.is_type(dictionary.get(keys[i]), Python.none()):
            print(keys[i], "is not in dictionary")
        else:
            print(keys[i], "is included")
```

输出：

```text
3 items
fruit is included
starch is included
protein is not in dictionary
```

## 参数化：编译时元编程


Python最令人惊叹的特性之一就是它可扩展的运行时元编程功能。这使得各种类型的库得以实现，并且提供了一种灵活且可扩展的编程模型，让全世界的Python程序员可以从中受益。不幸的是，这些功能也是有代价的：因为它们是在运行时评估的，它们直接影响底层代码的运行时效率。由于它们对IDE不熟悉，因此IDE功能（如代码补全）很难理解它们，从而无法改善开发者的体验。


在Python生态系统之外，静态元编程也是开发的重要组成部分，它使得开发新的编程范式和先进的库成为可能。在这个领域有许多先前技术的例子，有不同的权衡，例如：

1. 预处理器（例如Sass）C 预处理器、Lex/YACC 等可能是最重量级的它们是完全通用的，但在开发人员体验和工具集成方面表现最差。
1. 一些语言（如Lisp和Rust）支持（有时称为“卫生”）宏展开功能，可以实现语法扩展和降低样板代码，且具备更好的工具集成。
1. 一些较老的语言，如C++，具有非常大且复杂的元编程语言（模板），它们是运行时语言的对等物。这些显然很难学习，编译时间很长，而且错误信息不友好。
1. 一些语言（如Swift）以一流的方式将许多功能构建到核心语言中，以便为常见情况提供良好的人机交互，，但可能以牺牲一般性为代价。
1. 但一些像Zig这样的新语言将语言解释器集成到编译流程中，并允许解释器在编译时反映抽象语法树。这允许具有更好的可扩展性和普遍性的宏系统具有许多相同的功能。

我们需要先进的元编程系统提供的高抽象能力来完成Modular在AI、高性能机器学习内核和加速器方面的工作。我们需要高级零成本抽象、富有表现力的库以及大规模集成多种算法。我们希望图书馆开发人员能够像在Python中那样扩展系统，提供可扩展的开发平台。

也就是说，我们不愿牺牲开发人员的体验（包括编译时间和错误消息），也不愿意建立一个难以教授我们可以从这些以前的系统中学习，但也可以利用新技术来构建，包括MLIR和细粒度的语言集成缓存技术。


因此，Mojo支持将编译时元编程作为编译的单独阶段构建到编译器中，在解析、语义分析和IR生成之后它使用相同的主机语言来运行时程序，也用于元程序，并利用MLIR来表示和评估这些程序。

让我们来看看一些简单的例子。

**关于“参数”：**Python开发者通常会将“arguments”和“parameters”这两个词在“传递给函数的东西”方面相当互换使用。我们决定在Mojo中重新使用“parameter”和“parameter expression”来表示编译时的值，并继续使用“argument”和“expression”来指代运行时的值。我们决定这使我们能够围绕像“parameterized”和“parametric”这样的词进行编译时元编程。

## 定义参数化类型和函数

你可以通过在方括号中指定参数名称和类型（使用[PEP695语法](https://peps.python.org/pep-0695/)的扩展版本）来参数化结构参数值与参数值不同，参数值在编译时已知，这使得可以实现更高级别的抽象和代码重用，以及编译器的优化，比如自动调优（autotuning）等。


例如，让我们来看一下[SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data)类型，它代表硬件中的低级矢量寄存器，能够存储多个标量数据类型的实例。硬件加速器不断引入新的矢量数据类型，甚至CPU可能具有512位或更长的SIMD矢量。为了访问这些处理器上的SIMD指令，数据必须按照正确的SIMD宽度（数据类型）和长度（向量大小）。


但是，用Mojo内置的类型来定义所有不同的SIMD变体是不可行的。因此，Mojo的`SIMD`类型（定义为结构）在其方法中暴露了常见的SIMD操作，并使SIMD数据类型和大小值参这允许您直接将数据映射到任何硬件上的SIMD向量。以下是Mojo的`SIMD`类型的简化版本：

```auto
struct SIMD[type: DType, size: Int]:
    var value: … # Some low-level MLIR stuff here

    # Create a new SIMD from a number of scalars
    fn __init__(inout self, *elems: SIMD[type, 1]):  ...

    # Fill a SIMD with a duplicated scalar value.
    @staticmethod
    fn splat(x: SIMD[type, 1]) -> SIMD[type, size]: ...

    # Cast the elements of the SIMD to a different elt type.
    fn cast[target: DType](self) -> SIMD[target, size]: ...

    # Many standard operators are supported.
    fn __add__(self, rhs: Self) -> Self: ...
```

使用参数定义每个SIMD变体对于代码重用非常有用，因为`SIMD`类型可以静态地表达所有不同的向量变体，而不要求语言预定义每个变体

因为`SIMD`是一种参数化类型，它的函数中的`self`参数携带了这些参数——完整的类型名称是`SIMD[type, size]`。虽然用这种方式写出来（如`splat()`的返回类型所示）是有效的，但是这样会显得冗长，因此我们建议使用`Self`类型（来自PEP673）来实现类似`__add__`示例的功能。

## 参数重载


函数和方法可以根据参数签名进行重载。根据以下规则，按优先级顺序过滤候选项：

1. 候选人具有最少的隐式转换（在参数和参数中）。
1. 候选者没有可变参数。
1. 候选者没有可变参数。
1. 候选人参数签名最短。


如果在应用这些规则后有多个候选者，那么超载解析就会失败。例如：

```auto
@register_passable("trivial")
struct MyInt:
    """A type that is implicitly convertible to `Int`."""
    var value: Int

    @always_inline("nodebug")
    fn __init__(_a: Int) -> Self:
        return Self {value: _a}

fn foo[x: MyInt, a: Int](): pass

fn foo[x: MyInt, y: MyInt](): pass

fn bar[a: Int](b: Int): pass

fn bar[a: Int](*b: Int): pass

fn bar[*a: Int](b: Int): pass

fn waldo[a: Int](b: Int): pass

fn waldo[a: Int, T: AnyType](y: T): pass

fn parameterOverloads[a: Int, b: Int, x: MyInt]():
    # `foo[x: MyInt, a: Int]()` is called because it requires no implicit
    # conversions, whereas `foo[x: MyInt, y: MyInt]()` requires one.
    foo[x, a]()

    # `bar[x: Int](y: Int)` is called because it does not have variadic
    # arguments or parameters.
    bar[a](b)

    # `waldo[x: Int](y: Int)` is called because it has a shorter parameter
    # signature than `waldo[x: Int, T: AnyType](y: T)`.
    waldo[a](b)
```

## 使用参数化类型和函数

可以通过在方括号中传递值来实例化参数类型和函数。例如，对于上面的`SIMD`类型，`type`指定数据类型，`size`指定SIMD向量的长度（必须是2的幂）：

```auto
fn funWithSIMD():
    # Make a vector of 4 floats.
    let small_vec = SIMD[DType.float32, 4](1.0, 2.0, 3.0, 4.0)

    # Make a big vector containing 1.0 in float16 format.
    let big_vec = SIMD[DType.float16, 32].splat(1.0)

    # Do some math and convert the elements to float32.
    let bigger_vec = (big_vec+big_vec).cast[DType.float32]()

    # You can write types out explicitly if you want of course.
    let bigger_vec2 : SIMD[DType.float32, 32] = bigger_vec
```

注意，`cast()`方法还需要一个参数来指定你想要的类型（上面的方法定义期望一个`target`参数值）因此，就像`SIMD`结构是一个通用类型定义一样，`cast()`方法是一个泛型方法定义，根据参数值在编译时而不是运行时被实例化。

上面的`funWithSIMD()`函数展示了具体类型的使用（也就是说，它使用已知的类型值实例化`SIMD`），但参数的主要优势来自于定义参数化算法和类型(使用参数值的代码）的能力。例如，这是如何定义一个具有`SIMD`的参数化算法，它既具有类型又具有宽度的不变性：

```auto
fn rsqrt[dt: DType, width: Int](x: SIMD[dt, width]) -> SIMD[dt, width]:
    return 1 / sqrt(x)
```


注意`x`参数实际上是基于函数参数的`SIMD`类型。运行时程序可以使用参数的值，因为在运行时程序需要参数之前，参数会在编译时被解析（但编译时参数表达式无法使用运行时的值）。


Mojo编译器在参数类型推断方面也很聪明。注意，上述函数能够调用参数[`sqrt()`](https://docs.modular.com/mojo/MojoStdlib/Math.html#sqrt)函数，而无需指定参数——编译器推断参数的方式就好像你明确地写出了 `sqrt[type, simd_width](x)` 一样。此外，请注意，`rsqrt()`选择定义其第一个参数名为`width`，即使`SIMD`类型将其命名为`size`，也没有问题。

## 参数表达式只是Mojo代码。


参数表达式是任何在预期参数处出现的代码表达式（如`a+b`）。参数表达式支持操作符和函数调用，就像运行时代码一样，所有参数类型使用与运行时程序相同的类 (such as `Int` and `DType`)

因为参数表达式使用与运行时Mojo代码相同的语法和类型，所以您可以使用语言的许多“依赖类型”例如，您可能希望定义一个辅助函数来连接两个SIMD向量：

```auto
fn concat[ty: DType, len1: Int, len2: Int](
    lhs: SIMD[ty, len1], rhs: SIMD[ty, len2]) -> SIMD[ty, len1+len2]:
      ...

fn use_vectors(a: SIMD[DType.float32, 4], b: SIMD[DType.float16, 8]):
    let x = concat(a, a)  # Length = 8
    let y = concat(b, b)  # Length = 16
```


注意，结果长度是输入向量长度的总和，可以用一个简单的`+`操作来表达。例如，请看标准库中的[`SIMD.shuffle()`](https://docs.modular.com/mojo/MojoStdlib/SIMD.html#shuffle)方法：它接受两个输入SIMD值，一个向量混洗掩码作为列表，并返回一个与混洗掩码长度相匹配的SIMD结果。

## 强大的编译时编程


有时候，当需要编写具有控制流的编译时逻辑时，简单的表达式就显得不够用了。例如，Mojo `Math` 模块中的 `isclose()` 函数对整数使用精确相等，而对浮点数使用“接近”比较。你甚至可以进行编译时递归。下面是一个“树形减少”算法的例子，它可以递归地将向量中的所有元素求和为标量：

```auto
struct SIMD[type: DType, size: Int]:
    ...
    fn reduce_add(self) -> SIMD[type, 1]:
        @parameter
        if size == 1:
            return self[0]
        elif size == 2:
            return self[0] + self[1]

        # Extract the top/bottom halves, add them, sum the elements.
        let lhs = self.slice[size // 2](0)
        let rhs = self.slice[size // 2](size // 2)
        return (lhs + rhs).reduce_add()
```


这使用了`@parameter if`功能，它是一个在编译时运行的`if`语句。它要求其条件必须是一个有效的参数表达式，并确保只有`if`语句的活动分支才能被编译到程序中。

## Mojo类型只是参数表达式

而我们已经展示了如何在类型中使用参数表达式，类型注释本身可以是任意表达式（就像在Python中一样）。Mojo中的类型具有一种特殊的元类型，允许定义类型参数化的算法和函数。例如，你可以像这样定义一个类似C++ `std::vector` 的算法：

```auto
struct DynamicVector[type: AnyType]:
    ...
    fn reserve(inout self, new_capacity: Int): ...
    fn push_back(inout self, value: type): ...
    fn pop_back(inout self): ...
    fn __getitem__(self, i: Int) -> type: ...
    fn __setitem__(inout self, i: Int, value: type): ...

fn use_vector():
    var v = DynamicVector[Int]()
    v.push_back(17)
    v.push_back(42)
    v[0] = 123
    print(v[1])      # Prints 42
    print(v[0])      # Prints 123
```


注意`type`参数被用作`value`参数的正式类型，以及`__getitem__`函数的返回类型。参数允许`DynamicVector`类型根据不同的用例提供不同的API。有许多其他情况可以从更高级的用例中受益。例如，并行处理库定义了`parallel`算法，该算法并行执行闭包N次，每次从上下文中输入一个值。这个值可以是任何类型：字符串、数字、数组、对象甚至是函数。

```auto
fn parallelize[
    arg_type: AnyType,
    func: fn(Int, arg_type) -> None,
](rt: Runtime, num_work_items: Int, arg: arg_type):
    # Not actually parallel: see Functional.mojo for real impl.
    for i in range(num_work_items):
        func(i, arg)
```

因为`func`参数允许引用先前的`arg_type`参数，而且可以细化它的类型，所以这是可能的。

另一个重要的例子是变参泛型，在这里，一个算法或数据结构可能需要在一组异构类型上定义：

```auto
struct Tuple[*ElementTys: AnyType]:
    var _storage : *ElementTys
```

注意：我们还没有足够的元类型帮助程序，但我们将来应该可以编写类似的东西，尽管重载仍然是处理这些更好的方法

```auto
struct Array[T: AnyType]:
    fn __getitem__[IndexType: AnyType](self, idx: IndexType)
       -> (ArraySlice[T] if issubclass(IndexType, Range) else T):
       ...
```

## `别名`：命名参数表达式

想要编译时候的值得命名是很常见的。而`var`定义了一个运行时的值，`let`定义了一个运行时的常量，我们需要一种方法来定义一个编译时的临时值。Mojo使用`alias`声明来实现这一点。例如，`DType` 结构体使用别名为枚举器实现了一个简单的枚举，就像这样（实际的内部实现细节可能会有所不同）。

```auto
struct DType:
    var value : UI8
    alias invalid = DType(0)
    alias bool = DType(1)
    alias int8 = DType(2)
    alias uint8 = DType(3)
    alias int16 = DType(4)
    alias int16 = DType(5)
    ...
    alias float32 = DType(15)
```


这允许客户端自然地将`DType.float32`用作参数表达式（也可以用作运行时值）。注意，这是在编译时调用`DType`的运行时构造函数。

别名还可以用于类型：由于类型是编译时表达式，因此可以方便地做一些事情，如：

```auto
alias Float32 = SIMD[DType.float32, 1]
alias UInt8 = SIMD[DType.uint8, 1]

var x : Float32   # Float32 works like a "typedef"
```

像`var`和`let`一样，别名也遵守作用域，您可以像预期的那样在函数中使用局部别名。

顺便说一句，`None` 和 `AnyType` 都被定义为[类型别名](https://docs.modular.com/mojo/MojoBuiltin/TypeAliases.html)。

## 自适应编译/自动调节


Mojo参数表达式允许您编写可移植的参数算法，就像您在其他语言中所做的那样，但是，在编写高性能代码时，你仍然需要选择具体的参数值来使用。例如，在编写高性能数值算法时，您可能希望使用内存分块来加速算法，但要使用的维度高度依赖于可用的硬件功能、缓存大小、融合到内核中的内容以及许多其他微妙的细节。


由于典型机器的向量长度取决于数据类型，而像`bfloat16`这样的数据类型在所有实现中并不完全支持。Mojo通过在标准库中提供一个`autotune`函数来提供帮助。例如，如果你想编写一个与向量长度无关的算法来处理一个数据缓冲区，你可以这样写：

```auto
from Autotune import autotune

def exp_buffer_impl[dt: DType](data: ArraySlice[dt]):
    # Pick vector length for this dtype and hardware
    alias vector_len = autotune(1, 4, 8, 16, 32)

    # Use it as the vectorization length
    vectorize[exp[dt, vector_len]](data)
```


编译此代码的实例时，Mojo会对此算法的编译进行分叉，并通过衡量目标硬件上最佳实践来决定使用哪个。根据用户定义的性能评估器，它评估`vector_len`表达式的不同值，并选择最快的一个。因为它会单独测量和评估每个选项，所以它可能会为Float32选择一个不同于Int8的向量长度。这个简单的功能非常强大——超越了简单的整数常量——因为函数和类型也是参数表达式。


你可以通过提供性能评估器并使用[`search()`](https://docs.modular.com/mojo/MojoStdlib/Autotune.html#search)标准库函数来对`exp_buffer_impl()`的搜索进行`search()`函数接受一个评估器和一个分叉函数，并以参数结果返回由评估器选择的最快实现。

```auto
from Autotune import search

fn exp_buffer[dt: DType](data: ArraySlice[dt]):
    # Forward declare the result parameter.
    alias best_impl: fn(ArraySlice[dt]) -> None

    # Perform search!
    search[
      fn(ArraySlice[dt]) -> None,
      exp_buffer_impl[dt],
      exp_evaluator[dt] -> best_impl
    ]()

    # Call the selected implementation
    best_impl(data)
```


在这个例子中，我们将`exp_evaluator`作为性能评估器提供给搜索函数。性能评估器被调用时传入一个候选函数列表，应该返回最佳函数的索引。Mojo的标准库提供了一个`Benchmark`模块，可以用来计时函数。

```auto
from Benchmark import Benchmark

fn exp_evaluator[dt: DType](
    fns: Pointer[fn(ArraySlice[dt]) -> None],
    num: Int
):
    var best_idx = -1
    var best_time = -1
    for i in range(num):
        candidate = fns[i]
        let buf = Buffer[dt]()

        # Benchmark this candidate.
        fn setup():
            buf.fill_random()
        fn wrapper():
            candidate(buf)
        let cur_time = Benchmark(2).run[wrapper, setup]()

        # Track the index of the fastest candidate.
        if best_idx < 0:
            best_idx = i
            best_time = cur_time
        elif best_time > cur_time:
            best_idx = f_idx
            best_time = cur_time

    # Return the fastest implementation.
    return best_idx
```

自动调节具有指数级的运行时复杂度。它受益于Mojo编译器堆栈的内部实现细节（特别是MLIR，集成缓存和编译分发）。这是一个高级用户功能，需要不断的开发和迭代。

## “价值生命周期”：价值的诞生、生活和死亡

在这一点上，您应该理解Mojo函数和类型的核心语义和特性，因此我们现在可以讨论它们如何结合起来。

许多现有的语言都以不同的权衡来表达设计点：例如，C++非常强大，但经常被指责“默认值错误”，这导致了错误和功能不正常。Swift很容易使用，但是模型不太可预测，会经常复制值，而且还依赖于“ARC优化器”来提高性能。Rust从一开始就有很强的值所有权目标来满足其借用检查器，但是依赖于可移动的值，这使得表达自定义移动构造函数变得具有挑战性，并且可能对`memcpy`性能造成很大压力。在Python中，一切都是对类的引用，因此它从来不会遇到类型问题。


对于Mojo，我们从这些现有的系统中学习到了东西，我们的目标是提供一个非常强大的模型，同时仍然易于我们也不希望要求编译器构建“尽力而为”和难以预测的优化通过“足够聪明”。

为了探索这些问题，我们查看不同的价值分类和相关的Mojo功能，从底部开始构建一个全面的系我们在示例中以C++作为主要比较点，因为它是众所周知的，但如果其他语言提供了更好的比较点，我们也会进行参考。

## 不能实例化的类型

Mojo中最基本的类型是不允许创建它的实例的：这些类型根本没有初始化器，如果它们有析构函数，它将永远不会被调用（因为无法存在需要销毁的实例）：

```auto
struct NoInstances:
    var state: Int  # Pretty useless

    alias my_int = Int

    @staticmethod
    fn print_hello():
        print("hello world")
```


Mojo类型默认不提供构造函数、移动构造函数、成员初始化器或其他任何内容，因此无法创建此`NoInstances`。要获取它们，你需要定义一个`__init__`方法或使用一个合成初始化器的装饰器。如图所示，这些类型可以用作“命名空间”，因为即使无法实例化该类型的实例，也可以引用静态成员比如`NoInstances.my_int` or `NoInstances.print_hello()`

## 不可移动和不可复制的类型


如果我们在复杂程度上迈出一步，我们将到达可以实例化的类型，但一旦它们被固定到内存中的地址，就不能隐式地移动或赋值。这在实现原子操作类型（例如C++中的`std::atomic`）或其他类型时非常有用，其中内存地址是其标识并且对其目的至关重要

```auto
struct Atomic:
    var state: Int

    fn __init__(inout self, state: Int = 0):
        self.state = state

    fn __iadd__(inout self, rhs: Int):
        #...atomic magic...

    fn get_value(self) -> Int:
        return atomic_load_int(self.state)
```


这个类定义了一个初始化器，但没有复制或移动构造函数，因此一旦它被初始化，就永远不能被移动这是安全有用的，因为Mojo的所有权系统完全是“地址正确”的 - 当这个对象是在栈上初始化或作为其他类型的字段，它永远不需要移动。

请注意，Mojo的方法仅控制内置的移动操作，例如`a = b`复制和[`^` transfer operator](#owned-arguments)。你可以为自己的类型（比如上面的`Atomic`）使用一种有用的模式，即添加一个显式的`copy()`方法（非“双下这对于程序员知道安全时明确复制实例是很有用的。

## 唯一的“移动型”类型


如果我们在能力阶梯上再迈出一步，我们将遇到“唯一”类型 - C++中有很多示例，例如像`std::unique_ptr`这样的类型，或者拥有底层POSIX文件描述符的`FileDescriptor`类型。这种类型在像Rust这样的语言中很普遍，这些语言不鼓励复制，但“移动”是免费的。在Mojo中，可以通过定义`__moveinit__`方法来拥有一种独特的移动类型来实现这些移动。例如：

```auto
# This is a simple wrapper around POSIX-style fcntl.h functions.
struct FileDescriptor:
    var fd: Int

    # This is how we move our unique type.
    fn __moveinit__(inout self, owned existing: Self):
        self.fd = existing.fd

    # This takes ownership of a POSIX file descriptor.
    fn __init__(inout self, fd: Int):
        self.fd = fd

    fn __init__(inout self, path: String):
        # Error handling omitted, call the open(2) syscall.
        self = FileDescriptor(open(path, ...))

    fn __del__(owned self):
        close(self.fd)   # pseudo code, call close(2)

    fn dup(self) -> Self:
        # Invoke the dup(2) system call.
        return Self(dup(self.fd))
    fn read(...): ...
    fn write(...): ...
```


消费性移动构造函数（`__moveinit__`）拥有现有的`FileDescriptor`，并将其内部实现细节移动到新实例。因为`FileDescriptor`的实例可能存在不同的位置，它们可以逻辑上移动——偷取一个值的主体并移动到另一个位置。

以下是一个极其糟糕的例子，它会多次调用`__moveinit__`：

```auto
fn egregious_moves(owned fd1: FileDescriptor):
    # fd1 and fd2 have different addresses in memory, but the
    # transfer operator moves unique ownership from fd1 to fd2.
    let fd2 = fd1^

    # Do it again, a use of fd2 after this point will produce an error.
    let fd3 = fd2^

    # We can do this all day...
    let fd4 = fd3^
    fd4.read(...)
    # fd4.__del__() runs here
```


注意如何使用后缀`^`“转移”操作符在各个拥有它的值之间转移值的所有权，该操作符销毁先前的如果你熟悉C++，可以把传输操作符想象成`std::move`，但在这种情况下，我们可以看到，它能够移动事物而无需将它们重置为可被销毁的状态：在C++中，如果移动操作符未能更改旧值的`fd`实例，它将被关闭两次。


Mojo跟踪值的活力，并允许您定义自定义移动构造函数。这很少需要，但是当需要的时候功能非常强大。例如，像[`llvm::SmallVector type`](https://llvm.org/docs/ProgrammersManual.html#llvm-adt-smallvector-h)这样的类型使用“内联存储”优化技术，它们可能希这是一个众所周知的技巧，用于减少malloc内存分配器的压力，但这意味着“移动”操作需要自定义逻辑来更新指针


使用Mojo，这只需要实现一个自定义的`__moveinit__`方法即可。在C++中实现这一点也很容易（尽管在不需要自定义逻辑的情况下需要额外的样板代码），但在其他流行的内存安全的语言中实现起来很困难。

除此之外，Mojo编译器提供了良好的可预测性和控制能力，但也非常复杂。它保留了消除临时变量及相应的复制/移动操作的权利。如果这不适合您的类型，您应该使用像`copy()`这样的显式方法而不是双下划线方法。

## 支持“偷袭”的类型


内存安全语言的一个挑战是，它们需要提供一个可预测的编程模型，以便编译器能够跟踪，编例如，尽管编译器可以理解下面第一个例子中的两个数组访问是对不同数组元素的访问，，但对于第二个示例来说，通常很难对其进行推理。

```auto
std::pair<T, T> getValues1(MutableArray<T> &array) {
    return { std::move(array[0]), std::move(array[1]) };
}
std::pair<T, T> getValues2(MutableArray<T> &array, size_t i, size_t j) {
    return { std::move(array[i]), std::move(array[j]) };
}
```


这里的问题是，仅仅从上面的函数体来看，根本无法知道或证明`i`和`j`的动态值是不是相同的。尽管可以维护动态状态来跟踪数组的各个元素是否活动，但这往往会导致显著的运行时开销（即使不有多种处理方式，包括一些相当复杂的解决方案，这些方案并不总是容易学习。


Mojo采取务实的方法，让Mojo程序员可以在不必绕过其类型系统的情况下完成任务。如上所示，它不强制类型可复制、可移动或甚至可构造，但它确实希望类型能够表达其完整的约定，并且希望能够支持程序员从C++等语言中期望的流畅设计模式。这里的（众所周知的）观察是，许多对象具有可以在不需要禁用其析构函数的情况下“窃取”，这要么是因为它们具有“空状态”（例如可选类型或可空指针），要么是因为它们具有创建高效且销毁不需要额外操作的空值（例如，`std::vector`可以在其数据部分使用空指针）。

为了支持这些用例，[`^` transfer operator](#owned-arguments)支持任意的LValues，当应用于一个LValue时，它会调用“stealing move constructor”。这个构造函数必须设置新值为活跃状态，并且可以修改旧值，但它必须将旧值置于其析构函数仍可正常工作的状态。例如，如果我们想将我们的`FileDescriptor`放入一个向量中并移出它，我们可能会选择扩展它，知道`-1`是一个哨我们可以这样实现：

```auto
# This is a simple wrapper around POSIX-style fcntl.h functions.
struct FileDescriptor:
    var fd: Int

    # This is the new key capability.
    fn __moveinit__(inout self, inout existing: Self):
        self.fd = existing.fd
        existing.fd = -1  # neutralize 'existing'.

    fn __moveinit__(inout self, owned existing: Self): # as above
    fn __init__(inout self, fd: Int): # as above
    fn __init__(inout self, path: String): # as above

    fn __del__(owned self):
        if self.fd != -1:
            close(self.fd)   # pseudo code, call close(2)
```


注意，“偷窃移动”构造函数从现有值中获取文件描述符，并使该值发生变化，以使其析构函数不执。这种技术有权衡之处，并不是每种问题的最佳选择。我们可以看到，它为析构函数增加了一个（廉价的）分支，因为它必须检查哨兵情况。一般认为，使用这种类型来实现可空性是不好的做法，因为更通用的特性，比如`Optional[T]`类型，是处理这些更好的方式


此外，我们计划在Mojo本身中实现`Optional[T]`，而`Optional`需要此功能。我们也相信，库作者比语言设计者更懂他们的领域问题，并且通常更倾向于将对该领域的完全控制权交给库的作者。因此，您可以选择（但不必）以选择的方式使您的类型参与此行为。

## 可复制类型


下一个步骤是可复制的类型。可复制类型也非常常见 - 程序员通常期望字符串和数组可以复制，每个Python对象引用都是可复制的 - 通过赋值指针并更改引用计数。


有很多方法可以实现可复制类型。可以像Python或Java那样实现引用语义类型，在这种情况下，可以传播共享指针；可以使用不可变的数据结这些方法各有不同的权衡，Mojo认为，虽然我们希望有一些常见的集合类型，但我们也可以支持广泛的专业化类型，专注于特定的用例。


在Mojo中，你可以通过实现`__copyinit__`方法来实现这一点。以下是使用简单`String`在伪代码中的一个示例：

```auto
struct MyString:
    var data: Pointer[UI8]

    # StringRef is a pointer + length and works with StringLiteral.
    def __init__(inout self, input: StringRef):
        self.data = ...

    # Copy the string by deep copying the underlying malloc'd data.
    def __copyinit__(inout self, existing: Self):
        self.data = strdup(existing.data)

    # This isn't required, but optimizes unneeded copies.
    def __moveinit__(inout self, owned existing: Self):
        self.data = existing.data

    def __del__(owned self):
        free(self.data.address)

    def __add__(self, rhs: MyString) -> MyString: ...
```


这种简单的类型是一个指向使用旧式C API通过malloc分配的“以null结尾”字符串数据的指针。它实现了`__copyinit__`，它维护了每个`MyString`实例都拥有其底层指针并在销毁时释放它的不变性。这种实现建立在我们上面看到的技巧之上，实现了一个`__moveinit__`构造函数，它允许它在一些常见情况下完全你可以在这个代码序列中看到这种行为：

```auto
fn test_my_string():
    var s1 = MyString("hello ")

    var s2 = s1    # s2.__copyinit__(s1) runs here

    print(s1)

    var s3 = s1^   # s3.__moveinit__(s1) runs here

    print(s2)
    # s2.__del__() runs here
    print(s3)
    # s3.__del__() runs here
```


在这种情况下，你可以看到为什么需要拷贝构造函数：如果没有它，将`s1`值复制到`s2`将是一个错误——因为您不能同时拥有两个同一非可复制类型的活动实例。移动构造函数是可选的，但有助于将`s3`赋值：如果没有它，编译器将从s1调用拷贝这在逻辑上是正确的，但会增加额外的运行时开销。

Mojo热切地破坏价值，这使得它能够将复制+销毁对转换为单步操作，这可以比C++带来更好的性能，而不需要普遍的 `std::move` 的微观管理

## 轻松类型

最灵活的类型就是“bags of bits”。这些类型是“无关紧要”的，因为它们可以在不调用自定义代码的情况下复制、移动和销毁。这类类型可以说是最常见的基本类型，它们包围着我们：像整数和浮点值等都是微不足道的例子。从语言的角度来看，Mojo不需要特殊的支持，类型作者可以将这些东西实现为空操作，并允许内联函数将它们消除掉。


这种方法不太理想有两个原因：第一个原因是我们不希望在简单类型上定义一堆方法时出现样板代码，第二个原因是我们不希望在编译时产生并推动一堆函数调用带来的开销，然后这些函数调用最终被内联掉而变成无效。此外，还有一个相关的问题，即许多这些类型在另一方面也是简单的：它们很小，并且应该在 CPU 的寄存器中直接传递，而不是间接地通过内存传递。


因此，Mojo提供了一个结构装饰器，可以解决所有这些问题。使用`@register_passable("trivial")`装饰器可以实现一种类型，这告诉Mojo该类型应该是可复制和可移动的，但没有用户它还告诉Mojo优先将值传递到CPU寄存器中，这可能会带来效率的提升。

TODO：这个装饰器需要重新考虑。缺乏自定义逻辑的复制/移动/销毁逻辑和“在寄存器中传递”的可行性是完全不相关的问题，应该分开讨论。这个前面的逻辑应该被包含在一个更加通用的`@value("trivial")`装饰器中，它与`@register_passable`是相关的。

## @value装饰器


Mojo的方法（如上所述）提供了简单而可预测的钩子，使您能够正确表达像`Atomic`这样的复杂低级别东西。这对于控制和简单的编程模型来说很棒，但是我们编写的大多数structs都是其他类型的简单聚合，我们不想为它们编写大量的样板代码！为了解决这个问题，Mojo提供了一个`@value`装饰器，用于合成结构体的样板代码。`@value` 可以看作是 Python 的 `@dataclass` 的延伸，处理新的 `__moveinit__` 和 `__copyinit__` Mojo 方法。


`@value` 装饰器会查看你类型的字段，并生成缺失的成员。例如，考虑一个简单的结构：

```auto
@value
struct MyPet:
    var name: String
    var age: Int
```

Mojo会注意到您没有成员初始化器、移动构造函数或复制构造函数，并会按照您写的方式为您合成这些：

```auto
fn __init__(inout self, owned name: String, age: Int):
    self.name = name^
    self.age = age

fn __copyinit__(inout self, existing: Self):
    self.name = existing.name
    self.age = existing.age

fn __moveinit__(inout self, owned existing: Self):
    self.name = existing.name^
    self.age = existing.age
```

Mojo只有在不存在时才会合成这些内容，因此可以通过为其中一个或多个定义自己的版本来覆盖其行为。例如，希望使用默认的成员和移动构造函数，但又需要自定义拷贝构造函数，这是相当常见的。

**注意：**如果您的类型包含任何[move-only](#unique-move-only-types)字段，Mojo将不会生成拷贝构造函数，因为它无法拷贝这些字段。此外，`@value` 装饰器仅适用于其成员可复制和/或移动的类型。如果你的结构体中有像`Atomic`这样的东西，那么它可能不是一个值类型，你也不想要这些成员。

目前没有办法抑制特定方法的生成或定制生成，但如果有需求，我们可以向`@value`生成器添加参数来实现。

`__init__`的参数都作为`owned`参数传递，因为结构体拥有并存储该值。这是一个有用的微调优化，可以使用只能移动的类型。简单的类型，比如`Int`，也会作为拥有值传递，但是由于这对它们没有任何意义，为了清晰起见，我们省略了标记和转移操作符(`^`)。

## 析构函数的行为

在Mojo中，任何结构体都可以有一个析构函数（`__del__()`方法），当值的生命周期结束时（通常是值最后被使用的时刻），该析构函数会自动运行。例如，一个简单的字符串可能看起来像这样（伪代码）：

```auto
@value
struct MyString:
    var data: Pointer[UInt8]

    def __init__(inout self, input: StringRef): ...
    def __add__(self, rhs: MyString) -> MyString: ...
    def __del__(owned self):
        free(self.data.address)
```


Mojo使用“As Soon As Possible”（ASAP）策略，在每次调用后立即销毁像`MyString`这样的值（通过调用`__del__()`析构函数）。Mojo不会等到代码块结束时才销毁未使用的值。即使在像`a+b+c+d`这样的表达式中，Mojo也会立即销毁中间表达式，只要它们不再被需要了，它不会等到语句结束才运行。


Mojo编译器在值死亡时自动调用析构函数，并且在何时运行析构函数方面提供强有力的保证。Mojo使用静态编译器分析来推断你的代码，并决定何时插入析构函数的调用。例如：

```auto
fn use_strings():
    var a = MyString("hello a")
    var b = MyString("hello b")
    print(a)
    # a.__del__() runs here for "hello a"


    print(b)
    # b.__del__() runs here

    a = MyString("temporary a")
    # a.__del__() runs here because "temporary a" is never used

    other_stuff()

    a = MyString("final a")
    print(a)
    # a.__del__() runs again here for "final a"
```

在上面的代码中，你会看到`a`和`b`值在早期就已经创建，每一个值的初始化都会与一个析构函注意`a`被多次销毁——每次接收到新值时都会发生一次。


这可能会让C++程序员感到惊讶，因为它与C++在作用域结束时销毁值的[RAII模式](https://en.cppreference.com/w/cpp/language/raii)不同。Mojo也遵循一个原则，即在构造函数中获取资源，在析构函数中释放资源，但是，在Mojo中，及时销毁（eager destruction）相比于C++中基于作用域的销毁（scope-based destruction）有一些明显的优势：

+ Mojo方法消除了类型实现重新赋值运算符（如C++中的`operator =（const T&）`和`operator =（T&&）`）的需要，使类型定义更加容易，并且消除了一种概念。

+ Mojo不允许可变引用与其他可变引用或不可变借用重叠。它提供可预测的编程模型的一个主要方式是尽快使对象的引用消失，避免编译器认为一个值仍然存活并干扰另一个值的混乱情况，而用户对此并不清楚。

+ 在最后使用时销毁值与“移动”优化非常匹配，它将“复制+删除”操作转换为“移动”操作，这是对C++移动优化（如NRVO）的一种推广。

+ 在C++中，在作用域结束时销毁值可能会对一些常见的模式（如尾递归）造成问题，因为析构函数的这对于某些函数式编程模式来说可能是一个重大的性能和内存问题。


重要的是，Mojo的及时销毁在Python风格的`def`函数中也可以很好地工作，以提供细粒度的销毁保证（无需垃圾回收器）。请注意，Python实际上并没有提供函数以外的作用域，因此在Mojo中使用类似C++的作用域销毁会变得不那么有用。

**注意：**Mojo还支持Python风格的[`with`语句](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement)，它提供了更加有意义的范围访问


Mojo方法更类似于Rust和Swift的工作方式，因为它们都具有强大的值所有权跟踪功能，并提供内存安全性。一个区别是它们的实现需要使用[动态“drop flag”](https://doc.rust-lang.org/nomicon/drop-flags.html)——它们维护隐藏的shadow变量来跟踪值的状态以提供安全性。这些通常会被优化掉，但是Mojo方法完全消除了这种开销，使生成的代码更快，避免了歧义。

## 字段敏感的生命周期管理

此外，Mojo的终身分析完全兼容控制流，它也完全考虑字段敏感性（每个结构的字段都是独立跟踪的换句话说，Mojo单独跟踪“整个对象”是否完全初始化/销毁。

例如，在上面定义的`MyString`结构体的基础上，考虑以下代码：

```auto
struct TwoStrings:
    var str1: MyString
    var str2: MyString
    fn __init__(inout self): ...
    fn __del__(owned self): ...

fn use_two_strings():
    var ts = TwoStrings()
    # ts.str1.__del__() runs here

    other_stuff()

    ts.str1 = MyString("hello a")     # Overwrite ts.str1
    print(ts.str1)
    # ts.__del__() runs here
```


注意，`ts.str1`字段在被设置后立即被销毁，因为Mojo知道它将在下面被覆盖。您也可以在使用[[transfer operator](#owned-arguments)时看到此内容，例如：

```auto
fn consume_and_use_two_strings():
    var ts = TwoStrings()
    consume(ts.str1^)
    # ts.str1.__moveinit__() runs here

    # ts is now only partially initialized here!
    other_stuff()

    ts.str1 = MyString()  # All together now
    use(ts)               # This is ok
    # ts.__del__() runs here
```

注意，代码转移了`str1`字段的所有权：在`other_stuff（）`的持续时间内，`str1`字段完全未初始化，因为所然后，在`use()`函数使用`str1`之前，它会被重新初始化（如果不这样做，Mojo 将会拒绝代码并显示未初始化字段的错误）


Mojo对此的规则非常强大而且有意识地简单明了：字段可以被暂时转移，但是"整个对象"必须使用聚合类型的初始化器进行构造，并使用聚合析构函数进行销毁。这意味着无法仅通过初始化其字段来创建对象，也无法仅通过销毁其字段来拆除对象。例如，这段代码无法编译：

```auto
fn consume_and_use_two_strings():
    var ts = TwoStrings() # ts is initialized
    consume(ts.str1^)
    consume(ts.str2^) # Both members are transferred/destroyed
    # Error: cannot run the 'ts' destructor without initialized fields

    var ts2 : TwoStrings # ts2 type is declared but not initialized
    ts2.str1 = MyString()
    ts2.str2 = MyString()  # Both the member are initalized
    use(ts2) # Error: 'ts2' isn't fully initialized
```


我们可以允许这种模式发生，但我们拒绝这种做法，因为一个价值不仅仅是它的组成部分的总和。考虑一个包含POSIX文件描述符作为整数值的`FileDescriptor`：销毁整数（无操作）和销毁`FileDescriptor`（可能调用`close()`系统调用）。因此，我们要求所有全值初始化都必须通过初始化程序完成，并使用它们的全值析构函数进行销毁。

Mojo内部确实有一个与Rust的[`mem::forget`](https://doc.rust-lang.org/std/mem/fn.forget.html)函数等效的功能，它明确禁用了析构函数，并具有相应的内部功能来“标记”一个对象，但目前尚未公开供用户使用。这意味着用户无法直接使用这些功能，而是由Mojo内部使用。

## `__init__`中的字段生命周期


`__init__`方法的行为几乎和其他方法一样，只是有一点小的魔法：它知道对象的字段是未初始化的，但它认为整个对这意味着一旦所有字段都初始化完成，你就可以将`self`作为一个整体对象使用。

```auto
struct TwoStrings:
    var str1: MyString
    var str2: MyString

    fn __init__(inout self, cond: Bool, other: MyString):
        self.str1 = MyString()
        if cond:
            self.str2 = other
            use(self)  # Safe to use immediately!
            # self.str2.__del__(): destroyed because overwritten below.

        self.str2 = self.str1
        use(self)  # Safe to use immediately!
```

同样，Mojo中的初始化程序完全覆盖`self`是安全的，例如通过委托给其他初始化程序：

```auto
struct TwoStrings:
    var str1: MyString
    var str2: MyString

    fn __init__(inout self): ...
    fn __init__(inout self, cond: Bool, other: MyString):
        self = TwoStrings()  # basic
        self.str1 = MyString("fancy")
```

## `__del__`和`__moveinit__`中`owned`参数的字段生命周期


对于移动构造函数和析构函数的`owned`参数，存在一点最终的魔法。总结一下，这些方法是这样定义的：

```auto
struct TwoStrings:
    var str1: MyString
    var str2: MyString
    fn __init__(...)

    fn __moveinit__(inout self, owned existing: Self): ...
    fn __del__(owned self): ...
```

这些方法面临一个有趣但又晦涩难懂的问题：这两种方法都负责拆解`owned` `existing`/`self`值。也就是说，`__moveinit__()` 销毁 `existing` 的子元素以将所有权转移到新实例，而 `__del__()` 则实现了其 `self` 实例的删因此，他们都想拥有和转换`owned`值的元素，他们肯定不希望`owned`值的析构函数运行（在`__del__()`方法的情况，那将导致一个无限循环）。


解决这个问题，Mojo特别处理这两种方法，假设在任何从方法返回时，它们的整个值都被破坏了。这意味着在字段值被传输之前，整个对象可以被使用。例如，这个就像你期望的那样工作：

```auto
struct TwoStrings:
    var str1: MyString
    var str2: MyString
    fn __init__(...)
    fn __moveinit__(inout self, owned existing: Self): ...

    fn __del__(owned self):
        log(self)       # Self is still whole
        # self.str2.__del__(): Mojo destroys str2 since it isn't used

        consume(self.str1^)
        # Everything has now been transferred, no destructor is run on self.
```


一般情况下，你不需要考虑这个，但如果你的逻辑中有指向成员的内部指针，你可能需要在析构函数通过将`_`赋值为“discard”模式可以实现这一点：

```auto
fn __del__(owned self):
    log(self) # Self is still whole

    consume(self.str1^)
    _ = self.str2
    # self.str2.__del__(): Mojo destroys str2 after its last use.
```

在这种情况下，如果`consume()`隐式地指向`str2`中的某个值，这将确保`str2`在最后一次使用时，也就是通过`_`丢弃模式访问时，`str2`不会被销毁。

## 生命期

TODO：解释返回引用的工作原理，与参数相关的生命周期如何协调一致。这还没有启用。

## 类型特征


这是一个非常类似于Rust特性或Swift协议或Haskell类型类的功能。注意，这还没有实现。

## 高级/隐蔽的Mojo功能


本节描述了重要的高级用户功能，用于构建标准库的最底层。这一层堆栈中居住着需要经验丰富的编译器内部知识才能理解和有效利用的狭窄特性。

## @register_passable struct 装饰器

默认的处理值的模型是它们存储在内存中，因此它们具有标识，这意味着它们间接地传递给函数并从函数。这对于无法移动的类型非常有用，对于大型对象或具有昂贵的复制操作的事物来说是一个安全的默认值。然而，对于像单个整数或浮点数这样的小东西来说，它是低效的。


通过使用 `@register_passable` 装饰器，Mojo允许结构体选择通过寄存器而不是通过内存传递来解决这个问题。您将在标准库中的`Int`类型上看到这个装饰器：

```auto
@register_passable("trivial")
struct Int:
    var value: __mlir_type.`!pop.scalar<index>`

    fn __init__(value: __mlir_type.`!pop.scalar<index>`) -> Self:
        return Self {value: value}
    ...
```


基本的`@register_passable`装饰器不会改变类型的基本行为：它仍然需要有一个`__copyinit__`方法才能被复制，可能仍然有`这个装饰器的主要影响是在内部实现细节上：`@register_passable`类型通常会被传递到机器寄存器（取决于底层架构的具体细节。）

对于典型的Mojo程序员来说，这个装饰器只有几个可观察的效果：

1. `@register_passable` 类型不能保存不是 `@register_passable` 类型的实例。
1. `@register_passable`类型的实例没有可预测的身份，因此`self`指针是不稳定/可预测的（例如，每次在哈希表中）。
1. `@register_passable` 参数和结果可以直接被 C 和 C++ 访问，而不是通过指针传递。
1. 这种类型的`__init__`和`__copyinit__`方法隐式地是静态的（就像Python中的`__new__`一样），并且以值的形式返回结果而不是采用`inout self`


我们预计这个装饰器将被广泛应用于核心标准库类型，但对于一般应用程序级代码可以安全忽略。


上面的`Int`示例实际上使用了这个装饰器的“trivial”变体。它改变了上述描述的传递约定，但也禁止了拷贝和移动构造函数和析构函数（将它们合成为平凡函数。)

TODO：由于它也适用于内存类型，因此Trivial需要被解耦到其自己的装饰器中。

## `@always_inline`装饰器

`@always_inline("nodebug")`：同样的事情，但没有调试信息，因此您不会进入Int上的+方法。

## `@parameter` 装饰器

`@parameter` 装饰器可以放置在捕获运行时值的嵌套函数上，以创建“参数化”捕获闭包。variables.这是Mojo中一个不安全的特性，因为我们目前没有模拟引用捕获变量的生命周期。这个特性的一个特殊方面是它允许将捕获运行时值的闭包作为参数值传递。

## 魔术运算符


C++代码有许多神奇的操作符，它们与值生命周期相交，比如“placement new”、“placement delete”和“operator=”，它们可以重新分配给一个现有值。Mojo在使用其所有语言特性并在安全构造物上组合时是一种安全的语言，不过，如果使用堆栈，就涉及到C风格指针和普遍存在的不安全性。Mojo是一种务实的语言，由于我们对与C/C++互操作以及在Mojo本身实现安全构造（如String）都很感兴趣，因此我们需要一种表达不安全操作的方式。


Mojo标准库`Pointer[element_type]`类型在MLIR中用`!pop.pointer<element_type>`类型实现，我们希望在Mojo中实现这些与C++等效的不安全结构。最终，这些将迁移到指针类型的所有方法上，但在此之前，一些需要作为内置操作符公开。

## 直接访问MLIR


Mojo提供对MLIR方言和生态系统的完全访问权限。请查看[Mojo中的低级IR](https://docs.modular.com/mojo/notebooks/BoolMLIR.html)，学习如何使用`__mlir_type`、`__mlir_op`和`__mlir_type`构造。Mojo通过调用底层MLIR构造实现所有内置和标准库API，这样做，Mojo实际上相当于MLIR的语法糖。

