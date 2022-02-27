---
layout: single
classes: wide
title: "Demystifying C++'s constexpr, consteval, constinit keywords"
categories: posts
excerpt: "Exploring C++20"
tags: [C++, C++20, programming, constexpr, consteval, constinit ]
comments: true
published: false
share: true
---

C++20 introduces two new keywords: `consteval` and `constinit`. Before we explain these new keywords, let's review `constexpr` keyword, because these new keywords are extensions of the same idea.

## What is constexpr?

![consexpr cppreference definition](../../assets/images/constexpr_cpp_def.png)

> specifies the value of a variable or a function **can appear** in constant expressions.

Constant expressions are expressions that can be evaluated at **compile time**.

The keyword here is **can**. This means that `constexpr` variables or functions can be evaluated by the compiler at compile time, but don't have to be evaluated. The compiler will not throw an error if a `constexpr` variable or a function can not be evaluated at compile time. Using `constexpr` makes it possible to evaluate a variable/function at compile time, but does not guarantee it.

### Why and when to use `constexpr`
Simply stated, use `constexpr` to improve the performance of a program by reducing the number of function calls. Basically, this removes the overhead of a function call; it avoids pushing the original function's context to the stack and jumping to the new function and executing the new function and returning. This is useful for low level embedded software for hardware with limited resources, because it delegates some of the work to the compiler.

**Example**: Calculate the Fibonacci sequence for a given n

```c++
// A C++ program to demonstrate the use of constexpr
constexpr long int fib(int n)
{
    if (n <= 1)
        return n;

    return fib(n-1) + fib(n-2);
}
int main ()
{
    const long int answer = fib(30);
    return 0;
}
```

The assembly code generated from the above C++ program (using [Compiler Explorer](https://godbolt.org/z/jscveEq3f) `x86-64 gcc 11.2`) is:

[Code on Compiler Explorer](https://godbolt.org/z/jscveEq3f)


{% highlight C# %}
main:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], 832040
        mov     eax, 0
        pop     rbp
        ret
{% endhighlight %}

The value 30th Fibonacci number([832040](https://www.quora.com/What-is-the-30th-term-in-the-Fibonacci-series)) is calculated by the compiler(line 4) before the program runs.

If we remove the `constexpr` from `fib` function([Compiler Explorer link](https://godbolt.org/z/zvhb9Yefq)) then the compiler generates the assembly code:

{% highlight C# %}
fib(int):
        push    rbp
        mov     rbp, rsp
        push    rbx
        sub     rsp, 24
        mov     DWORD PTR [rbp-20], edi
        cmp     DWORD PTR [rbp-20], 1
        jg      .L2
        mov     eax, DWORD PTR [rbp-20]
        cdqe
        jmp     .L3
.L2:
        mov     eax, DWORD PTR [rbp-20]
        sub     eax, 1
        mov     edi, eax
        call    fib(int)
        mov     rbx, rax
        mov     eax, DWORD PTR [rbp-20]
        sub     eax, 2
        mov     edi, eax
        call    fib(int)
        add     rax, rbx
.L3:
        mov     rbx, QWORD PTR [rbp-8]
        leave
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     edi, 30
        call    fib(int)
        mov     QWORD PTR [rbp-8], rax
        mov     eax, 0
        leave
        ret
{% endhighlight %}

The difference between using `constexpr` and not using it is huge! Notice all the recursive calls to `fib`.

## What is consteval?
![consteval CPP Reference Definition](../../assets/images/consteval_def.png)

`consteval` declares **immediate functions**, functions which are evaluated at compile time to produce a constant. A `consteval` function is executed at compile time.

Unlike `constexpr`, `consteval` functions **must** always produce a compile time constant expression. The compiler will throw an error if it cannot evaluate a `consteval` function.

**Example:**

```c++
// constevalFunc.cpp
#include <iostream>
using std::cout;
using std::endl;

consteval long int fib(int n)
{
    if (n <= 1)
        return n;

    return fib(n-1) + fib(n-2);
}
int main ()
{
    cout << "fib(24): " << fib(24) << endl;     // 46368
    const int foo = 30;
    cout << "fib(foo): " << fib(foo) << endl;   // 832040

    int bar = 11;
    cout << "fib(bar): " << fib(bar) << endl; // Compiler Error
    return 0;
}
```
24 is a constant expression so the compiler is able to evaluate `fib(24)` into a constant expression.

`foo` is also a constant expression because of the `const` keyword, so compiler has no issues evaluating the call `fib(foo)` into a constant expression.

`bar` is not a constant, and its value can be changed, so the compiler throws an error (below).

![consteval compiler error](../../assets/images/consteval_compiler_error.png)


## What is constinit?
> constinit - asserts that a variable has static initialization, i.e. zero initialization and constant initialization, otherwise the program is ill-formed. 

`constinit` ensures that static variables (or `thread_local` variables)  are initialized at compile-time. This keyword is specific for variables with static storage duration or thread storage duration. This eliminates [static initialization order fiasco](https://isocpp.org/wiki/faq/ctors#static-init-order).

**Example**:
```c++
// Source: https://en.cppreference.com/w/cpp/language/constinit
const char *g() { return "dynamic initialization"; }
constexpr const char *f(bool p) { return p ? "constant initializer" : g(); }

constinit const char *c = f(true); // OK
// constinit const char *d = f(false); // error
```


## Summary
`constexpr` instructs the compiler that an expression or function can/may result in a compile time constant value. `consteval` instructs the compiler that a function must be evaluated at compile time. `constinit` initializes static variables in compile time.


**References**
- [When should you use constexpr capability in C++11](https://stackoverflow.com/questions/4748083/when-should-you-use-constexpr-capability-in-c11)
- [What is `constinit` in C++20?](https://stackoverflow.com/questions/57845131/what-is-constinit-in-c20)

