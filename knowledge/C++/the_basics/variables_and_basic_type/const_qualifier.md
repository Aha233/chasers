[TOC]

# C2.4 const限定符
有时候我们希望定义这样一种变量，它的值不能被改变。例如，用一个变量来表示缓冲区的大小。

为了满足这个要求，可以用关键字`const`对变量的类型加以限定：
```cpp
const int bufSize = 512;    //输入缓冲区
```

这样就ba`bufSize`定义成了一个常量。任何试图为`bufSize`赋值的行为都将引发错误：
```cpp
bufSize = 512;        //错误：试图向const对象写值
```

因为`const`对象一旦创建后其值就不能再改变，所以`const`对象必须初始化。一如既往，初始值可以是任意复杂的表达式：
```cpp
const int i = getSize();
const int j = 42;

const int k;                //错误：k是一个未经初始化的常量
```

**初始化和const**   
对象的类型决定了它所能执行的操作。与非`const`类型所能参与的操作相比，`const`类型的对象能完成其中大部分，但也不是所有操作都适合。主要的限制就是只能在`const`类型的对象上执行不改变其内容的操作。例如，`const int`和普通的`int`一样都能参与算数运算，也都能转换成一个布尔值，等等。

在不改变`const`对象的操作中还有一种就是初始化，如果利用一个对象去初始化另外一个对象，则它们是不是`const`都无关紧要：
```cpp
int i = 43;
const int ci = i;
int j = ci;
```

尽管`ci`是整型常量，但无论如何`ci`中的值还是一个整型数。`ci`的常量特征仅仅在执行改变`ci`的操作时才会发挥作用。

**默认状态下，const对象仅在文件内有效**  
当以编译初始化的方式定义一个`const`对象时，就如对`bufSize`的定义一样：  
```cpp
const int bufSize = 512;
```

编译器将在编译过程中把用到该变量的地方都替换成对应的值。也就是说，编译器会找到代码中所有用到过`bufSize`的地方，然后用`512`替换。

为了执行上述替换，编译器必须知道变量的初始值。如果程序包含多个文件，则每个文件用了`const`对象的文件都必须得能访问到它的初始值才行。要做到这一点，就必须在每个用到变量的文件中都有对它的定义。为了支持这一用法，同时避免对同一变量的重复定义，默认情况下，`const`对象被设定为仅在文件内有效。当多个文件中出现了同名的`const`变量时，其实等于在不同文件中分别定义了独立的变量。

某些时候有这样一种`const`变量，它的初始值不是一个常量表达式，但又确实有必要在文件中共享。这种情况下，我们不希望编译器为每个文件分别生成独立的变量。相反，我们想让这类`const`对象像其他（非常量）对象一样工作，也就是说，只在一个文件中定义`const`，而在其他多个文件中声明并使用它。

解决办法是，对于`const`变量不管是声明还是定义都添加`extern`关键字，这样只需定义一次就可以了：  
```cpp
//file_1.cc 定义并初始化了一个常量，该常量能被其他文件访问
extern const int bufSize = fcn();
//file_1.h 头文件
extern cosnt int bufSize;    //与file_1.cc中定义的bufSize是同一个
```

## const的引用
可以把引用绑定到`const`对象上，就像绑定到其他对象上一样，我们称之为**对常量的引用（reference to const）**。与普通引用不同的是，对常量的引用不能被用作修饰它所绑定的对象：
```cpp
const int ci = 2048;
const int &ri = ci;    //正确：引用及其对应的对象都是常量
r1 = 42;               //错误：r1 是对常量的引用
int &r2 = ci;          //错误：试图让一个非常量引用指向一个常量对象
```

### 对const的引用可能引用一个并非const的对象
必须认识到，常量引用仅对引用可参与的操作做出了限定，对于引用的对象本身是不是一个常量未做限定。因为对象也可能是一个非常量，所以允许通过其他途径改变它的值：
```cpp
int i = 42;
int &r1 = i;
const int &r2 = i;
r1 = 0;
r2 = 0;                //错误：r2是一个常量引用
```

## 指针和const
与引用一样，也可以令指针指向常量或非常量。类似于常量引用，**指向常量的指针（pointer to const）** 不能用于改变其所指向对象的值。要想存放常量对象的地址，只能使用指向常量的指针：
```cpp
const double pi = 3.14;
double *ptr = &pi;             //错误：ptr是一个普通指针
const double *cptr = &pi;
*cptr = 42;                    //错误：不能给*cptr赋值
```

### const指针
指针是对象而引用不是，因此就像其他对象类型一样，允许把指针本身定位常量。  
**常量指针（const pointer）** 必须初始化，而且一旦初始化完成，则它的值就不能再改变了。
```cpp
int errNumb = 0;
int *const curErr = &errNumb;
const double pi = 3.14;
const double *const pip = &pi;
```

理解这些声明的最有效办法就是从右向左读。  
在`int *const curErr = &errNumb`中，离`curErr`最近的符号是`const`，意味着`curErr`本身是一个常量对象，对象类型由声明符的其余部分确定。声明符的下一个符号是`*`，意思是`curErr`是一个常量指针。最后，该语句的基本数据类型部分确定了常量指针指向一个`int`对象。

## 顶层const
如前所述，指针本身是一个对象，它又可以指向另外一个对象。因此，指针本身是不是常量以及指针所指向的是不是一个常量就是两个相互独立的问题。  
用名词**顶层const（top-level const）** 表示指针本身是一个常量。  
用名词**底层const（low-level const）** 表示指针所指的对象是一个常量。

更一般的，顶层const可以表示任意的对象是常量。  
底层const则与指针和引用等复合类型的基本类型部分有关。

比较特殊的是，指针类型既可以是顶层const也可以是底层const。

## constexpr和常量表达式
**常量表达式（const expression）** 是指值不会改变并且在编译过程中就能得到计算结果的表达式。显然，字面值属于常量表达式，用常量表达式初始化的`const`对象也是常量表达式。

### constexpr变量
在一个复杂的系统中，很难（几乎不可能）分辨一个初始值到底是不是一个常量表达式。

C++ 11规定，允许将变量声明为**constexpr**类型以便由编译器验证变量的值是否是一个常量表达式。