---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/
# image: https://cooltext.com/

title: "C++ Diary #3 | Move Semantics - Rvalue Reference"
subtitle: ""
summary: "Understanding Move Semantics Part I: A Deep Dive into Rvalue References"
authors:
- admin
tags:
- cpp
categories: ["C++"]
date: 2024-08-06
lastmod: 2024-08-06
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---




## Background

The first time I learned about move semantics was when I read Scott Meyers' book *Effective Modern C++*. Although I thought I had a basic understanding, I only remembered the famous statement that **`std::move` doesn't actually move anything** after many years.

Recently, I started using move semantics more frequently and noticed that others were also struggling to understand why and how to use it at very beginning like me. So, I decided it was time to invest more effort into educating myself and sharing my learning experience.

## From `std::move` to Rvalue Reference

Initially, I was confused when I searched for `std::move` on [cppreference](https://en.cppreference.com/w/cpp/utility/move), as it mentioned, "std::move is exactly equivalent to a `static_cast` to an rvalue reference type."

Here’s a simple implementation of `std::move`:

```cpp
template <typename T>
typename std::remove_reference<T>::type&& move(T&& arg) noexcept {
    return static_cast<typename std::remove_reference<T>::type&&>(arg);
}
```

There are two key takeaways:

1. **`T&& t`** is a Universal Reference (or Forwarding Reference) because `T` is a template parameter. This can be misleading if you are unfamiliar with Universal References, as it might seem like it only accepts an rvalue reference.
2. It returns an *rvalue reference* to arg.

If you’re not familiar with Universal References or rvalue references, don't worry. Let's start with the basics and discuss lvalues, rvalues, and their references in this post.

## Lvalues and Rvalues

In C++, the concepts of `lvalue` and `rvalue` are fundamental to understanding expressions and their evaluation. Here’s a clear breakdown:

- An `lvalue` represents an object that occupies some identifiable location in memory. It can appear on **both the left and right sides** of an assignment operator. You can use the `&` operator to obtain the address of an `lvalue`.
- An `rvalue` represents a temporary value that does not persist beyond the expression in which it is used. It can **only appear on the right side** of an assignment operator. `Rvalue` are often temporary objects created during expression evaluation or literal constants.
- Every expression is either an `lvalue` or an `rvalue`.

<aside>
💡 Key Principles:

- An `lvalue` is stored in an object and has a persistent state. An `rvalue` is either a literal constant or a temporary object created during the evaluation of an expression and does not have a persistent state.
- An `lvalue` can be used in contexts where an `rvalue` is required. Conversely, an `rvalue` cannot be used in contexts where an `lvalue` is required.
</aside>

Let’s take a look of an example:

```cpp
int GetValue() {
    return 10;
}

int& GetValue2() {
    static int value = 10;
    return value;
}

int i = 42;
int a = i;  // both a and i are lvalues
int *p = &i;  // p is also lvalue

int j = GetValue();  // j is lvalue, and GetValue() is rvalue
int *p2 = &GetValue();  // ERROR，cannot take the address of an rvalue!
j = 42;  // ok, 42 rvalue (a literal constant)

GetValue2() = 42;  // ok, GetValue2() is lvalue
int *p1 = &GetValue2();  // ok, because GetValue2() is lvalue
```

## Lvalue Reference

In C++, references can be defined using the `&` symbol. When an `lvalue` is also a reference, it is called an `lvalue reference`. **Lvalue references are the most common type of reference**, so when we generally talk about "object reference", we are referring to `lvalue reference`. For example:

```cpp
std::string s;
std::string& sref = s;  // sref is an lvalue reference
```

You might wonder if you can take an `lvalue reference` to an `rvalue`.

<aside>
💡 Key Principles:

- You cannot take an `lvalue reference` to an `rvalue`.
- You can only have an `lvalue reference` to an `lvalue`.
</aside>

If it were possible to take an `lvalue reference` to an rvalue, it would create a problem: how would you modify the value of the rvalue? Since references can be updated later, and the reference to rvalue do not have a retrievable memory address, modifying an rvalue would be impossible.

However, `const lvalue references` are different because constants cannot be modified, eliminating the issue above:

```cpp
// Error! std::string() creates a temporary object, which is an rvalue
std::string& r = std::string("TEST");
// Valid
const std::string& r = std::string("TEST");  
```

Another example, which also explains why `const lvalue reference` is useful:

```cpp
void PrintName1(std::string& name) {
    std::cout << name << std::endl;
}

void PrintName2(const std::string& name) {
    std::cout << name << std::endl;
}

int main() {
    // both first_name and last_name are lvalue
    std::string first_name = "Foo";
    std::string last_name = "Bar";
    // ERROR! 
    // Because "first_name + last_name" is actually an rvalue (temporary) 
    PrintName1(first_name + last_name);
    // Correct!  
    PrintName2(first_name + last_name);
}
```

## Rvalue Reference (since C++11)

Rvalue references and the move semantics are among the most powerful features introduced in C++11. Previously, we discussed that lvalues (non-const) can be modified (assigned to), but rvalues cannot. **However, with C++11's introduction of rvalue references, this limitation is lifted, allowing us to obtain references to rvalues and modify them.**

Let’s follow the example from lvalue reference section:

```cpp
void PrintName3(const std::string&& name) {
    std::cout << name << std::endl;
}

PrintName3(firstName + lastName);  // Correct!
PrintName3(fullName);  // ERROR! Because PrintName3() only accept rvalue now!
```

Rvalue references can only bind to rvalues, which are either literal constants or temporary objects that soon to be destroyed. 

It means that **code accepting and using rvalue references can freely take over the resources of the referenced object without worrying about data in other parts of the code**. In other words, it **transfers ownership** of the assets and properties of an object directly.

Rvalue references enable move semantics and perfect forwarding. We will discuss rvalue references and `std::move` further in the next post.

## References

1. [谈谈 C++ 中的右值引用](https://liam.page/2016/12/11/rvalue-reference-in-Cpp/)
2. [C++ 11 右值引用以及std::move](https://blog.csdn.net/luotuo44/article/details/46779063)
3. [lvalues and rvalues in C++ - The Cherno](https://www.youtube.com/watch?v=fbYknr-HPYE)
4. [std::move and the Move Assignment Operator in C++ - The Cherno](https://www.youtube.com/watch?v=OWNeCTd7yQE)
5. [Hidden Features and Traps of C++ Move Semantics](https://www.youtube.com/watch?v=sACa3Ax7YB4)
6. [What is move semantics?](https://stackoverflow.com/questions/3106110/what-is-move-semantics)