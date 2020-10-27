---
tital: C++ static 与 const
tags: 面试问题
---

#### static 关键字

1. 将具有 **外部链接** 属性的变量/函数转换为 **内部链接** 

   全局变量和函数的链接属性默认为外部链接（对其他文件可见）

   我们在 `test.cpp` 文件中定义全局变量`global`，并声明函数 `func()`

   *test.cpp*

   ```cpp
   #include<iostream>
   
   using namespace std;
   
   void func();
   
   int global = 0;
   
   
   int main()
   {
   	func(); // 调用 test2.cpp 中定义的 func 函数
   }
   ```

   在`test2.cpp`文件中定义函数`func()`并声明变量`global`  

   *test2.cpp*

   ```cpp
   #include<iostream>
   
   extern int global;
   
   void func()
   {
       // 使用 test.cpp 中定义的 global 变量
   	std::cout << "global int test2.cpp: " << global << std::endl; 
   }
   ```

   编译两个文件并运行，得到输出：

   ```
   global int test2.cpp: 0
   ```

   现在我们使用 `static` 声明函数和变量：

    ```cpp
   static int global = 0;
   
   static void func()
   {
   	std::cout << "global int test2.cpp: " << global << std::endl;
   }
    ```

   编译发现报错：

   ```
   错误	LNK2019	无法解析的外部符号 "void __cdecl func(void)" (?func@@YAXXZ)，函数 _main 中引用了该符号	
   
   错误	LNK2001	无法解析的外部符号 "int global" (?global@@3HA)	
   ```

   此时 `global` 对 `test2.cpp` 是不可见的；`func()` 函数对 `test.cpp` 是不可见的

   

2. 将具有 **自动存储期限** 的变量转换为 **静态存储期限** 

   auto变量（函数局部变量）都是在栈内存区存放，函数结束后就自动释放。但是全局的和函数内定义的static变量都是存放在数据区的，且只存一份，只在整个程序结束后才自动释放。

   ```cpp
   void func()
   {
       // static 类型变量只在程序第一次执行到定义时初始化
   	static int i = 0; 
   	int j = 0;
   	cout << "i = "<< i++ << " j = " << j++ << endl;
   }
   int main()
   {
   	func();
   	func();
   	func();
   }
   ```

   输出：

   ```
   i = 0 j = 0
   i = 1 j = 0
   i = 2 j = 0
   ```

   

3. C++ 中 **静态成员变量** ：整个类共享。必须在类外定义

   ```cpp
   class A
   {
   private:
   	int _i;
   	static int _si;
   
   public:
   	static int _hi;
   	static void print();
   	void show();
   };
   
   // 只有定义时可以使用 类名:: 访问到，其他时候不可以
   int A::_si = 10;
   int A::_hi = 20;
   
   int main()
   {
   	A a;
   
   	// 不在类外定义，会报连接错误
   	cout << A::_hi << endl;
   	cout << a._hi << endl;
   	// cout << A::_si << endl; // ERROR _si 不可访问
   }
   ```

   **注意** ：如果 `_hi` 的类型为 `static const` 那么可以直接在类内初始化

   ```cpp
   static const int _hi = 10;
   ```

   **静态成员函数** ：只能调用静态成员函数，使用静态成员变量（没有 this 指针）

   ```cpp
   class A
   {
   private:
   	int _i;
   	static int _si;
   
   public:
   	static  int _hi;
   	static void print();
   	void show();
   };
   
   int A::_si = 10;
   int A::_hi = 20;
   
   void A::print()
   {
   	cout << "_si = " << _si << endl;
   	// cout << "i = " << _i << endl; // ERROR 静态成员函数只能只能调用静态成员函数，使用静态成员变量
   }
   
   void A::show()
   {
   	cout << "i = " << _i << endl;
   	cout << "_si = " << _si << endl; // OK
   }
   
   int main()
   {
   	A a;
   	
   	A::print();
   	// A::show(); // ERROR 非静态成员函数不能使用 类名 + :: 访问
   	a.print();
   	a.show();
   }
   ```

   

参考文章：

https://www.jianshu.com/p/0b2d9679a9f2



static 变量的初始化顺序：

以下是 Stackoverflow 上的一个回答原文，链接在下面

> Before any function in a translation unit is executed (possibly after `main` began execution), the variables with static storage duration (namespace scope) in that translation unit will be "constant initialized" (to `constexpr` where possible, or zero otherwise), and then non-locals are "dynamically initialized" properly *in the order they are defined in the translation unit* (for things like `std::string="HI";` that aren't `constexpr`). Finally, function-local statics will be initialized the first time execution "reaches" the line where they are declared. All `static` variables all destroyed in the reverse order of initialization.

全局作用域的 `static` 变量在本翻译单元任何函数执行前初始化

函数（局部）作用域的 `static` 变量在程序首次执行到该变量声明处出进行初始化

所有的 `static` 变量以初始化的相反顺序销毁。

对没有使用常量表达式初始化的 static 变量，可以将其放入函数中进行初始化：

> The easiest way to get all this right is to make all static variables that are not `constexpr` initialized into function static locals, which makes sure all of your statics/globals are initialized properly when you try to use them no matter what, thus preventing the [static initialization order fiasco](https://stackoverflow.com/questions/3035422/static-initialization-order-fiasco).

```cpp
T& get_global() {
    static T global = initial_value();
    return global;
}
```

参考问题：

https://stackoverflow.com/questions/15235526/the-static-keyword-and-its-various-uses-in-c?r=SearchResults

**SIOF** （static initialization order fiasco）问题

```cpp
//file1.cpp
extern int y;
int x=y+1;

//file2.cpp
extern int x;
int y=x+1; 
```

问：`x` 和 `y` 会获得什么值？

参考问题：

https://stackoverflow.com/questions/3035422/static-initialization-order-fiasco

原回答如下：

>The initialization steps are given in 3.6.2 "Initialization of non-local objects" of the C++ standard:
>
>Step 1: `x` and `y` are zero-initialized before any other initialization takes place.
>
>Step 2: `x` or `y` is dynamically initialized - which one is unspecified by the standard. That variable will get the value `1` since the other variable will have been zero-initialized.
>
>Step 3: the other variable will be dynamically initialized, getting the value `2`.



#### const 关键字

《Effective C++》明确指出：**尽可能地使用 const** （Use const whenever possible）

**1. const 与指针**

```cpp 
char greeting[] = "Hello";
char* p = greeting;       // non-const data,non-const pointer
const char* p = greeting; // const data,non-const pointer
char const* p = greeting; // const data,non-const pointer
char* const p = greeting; // non-const data,const pointer
const char* const p = greeting; // const data,const pointer
```

如果关键字 `const` 出现在 `*` 左侧，表示被指物是常量；如果关键字 `const` 出现在 `*` 右侧，表示指针自身是常量。

**2. STL 迭代器**

STL 的 `const_iterator` 表示迭代器所指物不可改变（const T*）：

```cpp
#include<vector>

int main()
{
	vector<int> v{ 1, 2, 3 };

	const vector<int>::iterator it = v.begin();
	*it = 10; // OK
	it++;	  // ERROR

	vector<int>::const_iterator cit = v.begin();
	*cit = 20; // ERROR
	cit++;	   // OK
}
```

**3. 修饰函数返回值和函数参数**

考虑有理数的 `operator*` 声明式：

```cpp
class Rational{...}
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

如果不返回 const，客户就可以实现这样的暴行：

```cpp
Rational a, b, c;
(a * b) = c;
```

将函数返回值或者参数修饰为 `const`，可以避免客户或函数内部修改该变量。

**4.修饰函数自身（成员函数）**

```cpp
class TextBlock
{
public:
    ...
    const char& operator[](std::size_t pos) const
    {return text[pos];}
    char& operator[](std::size_t pos)
    {return text[pos];}
private:
    std::string text;
}
```

注意 `non-const` 函数返回类型是 `reference to char`，不是 `char`。如果返回类型是 char，下面的语句就无法通过编译：

```cpp
text[0] = 'x';
```

如果函数返回类型是个内置类型，那么改动函数返回值从来就不合法。

**bitwise constness 与 logical constness**

bitwise const 阵营的人认为成员函数只有在不更改对象的任何成员变量（static 除外）时才可以说是 const。也就是不更改对象内的任何一个 bit 。请看下面的类：

```cpp
#include<string.h>
class CTextBlock
{
private:
	char* _pText;
public:
	CTextBlock()
		:_pText(new char[10])
	{
		strcpy(_pText, "Hello");
	}
	char& operator[](int pos)const
	{
		return _pText[pos];
	}
};

int main()
{
	CTextBlock ctb;
	ctb[0] = 'A';
}
```

虽然修改了指针所指之物，但是只有指针隶属于对象，而指针所指之物并非对象成员，称此函数为 bitwise const 不会引发编译器异议。

logical constness 主张一个 const 成员函数可以修改它所处理的对象内的某些 bits，但只有客户端侦测不出的情况下才得如此。例如你的 `CTextBlock class` 有可能高速缓存（cache）文本区块的长度以便应付询问： 

```cpp
class CTextBlock
{
private:
	char* _pText;
	std::size_t _textLength; // 最近一次计算的文本区块长度
	bool _lengthIsValid;     // 目前长度是否有效
public:
	...
	std::size_t length() const;
};
std::size_t CTextBlock::length() const
{
	if (!_lengthIsValid)
	{
		_textLength = std::strlen(_pText); // ERROR
		_lengthIsValid = true;			   // ERROR 不能在 const 成员函数内修改成员变量的值 
	}
	return _textLength;
}
```

length 的实现不是 bitwise const，但是编译器不同意修改，坚持 bitwise const，怎么办？

解决办法：使用 `mutable` 释放掉 `non-static` 成员变量的 bitwise const 约束：

```cpp
class CTextBlock
{
private:
	char* _pText;
	mutable std::size_t _textLength; // 这些成员变量总是可以被更改，即使在 const 成员函数内
	mutable bool _lengthIsValid;     
public:
	CTextBlock()
		:_pText(new char[10])
	{
		strcpy(_pText, "Hello");
	}
	std::size_t length() const;
};
std::size_t CTextBlock::length() const
{
	if (!_lengthIsValid)
	{
		_textLength = std::strlen(_pText); // OK
		_lengthIsValid = true;
	}
	return _textLength;
}
```

**在 const 和 non-const 成员函数中避免重复**

```cpp
class CTextBlock
{
private:
	std::string _pText;
public:
	const char& operator[](std::size_t pos) const
	{
		...//边界检查
		...//志记数据访问
		...//检验数据完整性
		return _pText[pos];
	}

	char& operator[](std::size_t pos)
	{
		...//边界检查
		...//志记数据访问
		...//检验数据完整性
		return _pText[pos];
	}
};
```

可以看到 const 和 non-const 成员函数功能完全相同，唯一不同就是 const 成员函数的返回类型多了一个 const 资格修饰。代码重复会带来编译时间，维护，代码膨胀等问题。

当然，可以将 边界检查等功能移到另一个成员函数中（访问类型为 private），但是还是会重复一些代码。

这里有一种选择是 **转型** ，用 non-const 成员函数调用 const 成员函数。

```cpp
class CTextBlock
{
private:
	std::string _pText;
public:
	const char& operator[](std::size_t pos) const
	{
		...//边界检查
		...//志记数据访问
		...//检验数据完整性
		return _pText[pos; ]
	}

	char& operator[](std::size_t pos)
	{
		// 用 static_cast 将 *this 转换为 const CTextBlock& 后调用 const operator[] 函数
		// 否则 会调用 non-const operator 函数，引发递归
		// 然后用 const_cast 去掉 const operator[] 函数的返回值的 const 约束
		return const_cast<char&>(
			static_cast<const CTextBlock&>(*this)[pos]);
	}
};
```

是否可以用 const 成员函数调用 non-const 成员函数？

如果要这样调用，首先我们要使用 `const_cast<CTextBlock&>(*this)` 去掉 *this 的 const 性质，这是乌云笼罩的清晰前兆。



参考资料：

《Effective C++》