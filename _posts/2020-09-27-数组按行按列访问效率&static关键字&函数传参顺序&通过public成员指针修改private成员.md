---
title: 数组按行按列访问效率&static关键字&函数传参顺序&通过public成员指针修改private成员
tags: 面试问题
---





#### 二维数组按行逐行访问快还是按列逐列访问快

**如果二位数组在内存中是连续的** 

> 比如二维数组是栈上的或者是一次 `new/malloc` 申请来的

参考文章：

https://blog.csdn.net/Shuffle_Ts/article/details/89420651

cache 一般按块来读取内存（连续内存）。如果二维数组比较小，可以一次载入缓存中，那可能按行或按列访问效率差别不大。但如果数组比较大，缓存无法一次载入整个数组（比如一次只能载入 1 行），这时，按行访问会更快。

**如果二维数组在内存中是不连续的**

> 二维数组的每一行都是调用一次 `new/malloc` 申请来的

参考问题：

https://stackoverflow.com/questions/17259877/1d-or-2d-array-whats-faster/17260533#

中的第一个回答。

回答中的有个图片可能不好加载，在这里给出：

![](https://hairrrrr.github.io/assets/2020-09-27-1.PNG)

这种情况相比于内存连续的数组，劣势在于：

- 多次申请/释放内存
- 内存的额外开销
- 内存泄漏的风险

如果出现下面的情况：

- 矩阵很大而且是稀疏的（有的行是可以使用 `nullptr` 来表示的）
- 每行的列数不同

使用这种数组是比较好的选择。



#### static 关键字

1. 将具有 **外部链接** 属性的变量/函数转换为 **内部链接** 

2. 将具有 **自动存储期限** 的变量转换为 **静态存储期限** 

3. C++ 中 **静态成员变量** ：整个类共享。必须在类外定义

   **静态成员函数** ：只能调用静态成员函数，使用静态成员变量（没有 this 指针）

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



#### 函数传参顺序

**例 1**

如果给你下面的程序，会输出什么？

```cpp
int main()
{
	int i = 10;

	printf("%d %d %d %d\n", i++, ++i, i, i++);

	return 0;
}
```

输出：

```
12 13 13 10
```

参考文章：

https://www.cnblogs.com/easonliu/p/4224120.html 

**例 2**

```c
void foo(int x, int y, int z)
{
	printf("x = %d at [%X]\n", x, &x);
	printf("y = %d at [%X]\n", y, &y);
	printf("z = %d at [%X]\n", z, &z);
}

int main(int argc, char* argv[])
{
	foo(100, 200, 300);
	return 0;
}
```

输出：

```
x = 100 at [113FBA0]
y = 200 at [113FBA4]
z = 300 at [113FBA8]
```



[C语言中函数参数为什么是由右往左入栈的？](https://blog.csdn.net/testcs_dn/article/details/48876771?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

**例 3**

I did an experiment:

```cpp
class A {
public:
  A() {}
  A(const A& a) {
    printf("A - %p\n", this);
  }
};

class B {
public:
  B() {}
  B(const B& b) {
    printf("B - %p\n", this);
  }
};

void func(A a, B b) {}

int main() {
  A a;
  B b;
  func(a, b);
  return 0;
}
```

The output is:

```
B - 0x7fff636e2c48
A - 0x7fff636e2c50
```

Since the parameters are pushed from right to left, why B's address is lower than A's? Confused. (Stack starts from the higher address).



参考问题：

https://stackoverflow.com/questions/8496384/parameter-push-order-of-a-c-function-call-does-not-reflect-the-parameters-add?r=SearchResults



#### 是否可以通过获取类中 public 类型的成员变量的地址来修改 private 成员变量的值

可以，但是要注意 **内存对齐** 

```cpp
#include<iostream>

using namespace std;

class A
{
	char ch;
public:
	int i;
private:
	double d;
	
public:
	void show() const
	{
		cout << ch << " " << d << endl;
	}
};

int main()
{
	A a;

	int* pi = &a.i;

	// 要考虑内存对齐
	// ch 和 i 之间有 3 个字节的 padding
	char* pch = (char*)(pi - 1);
	*pch = 'a';

	double* pd = (double*)(pi + 1);
	*pd = 3.14;

	a.show();
}
```









