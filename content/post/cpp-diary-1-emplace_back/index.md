---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/
# image: https://cooltext.com/

title: "C++ Diary #1 | emplace_back vs. push_back"
subtitle: ""
summary: "Pros and cons of emplace_back"
authors:
- admin
tags:
- cpp
categories: ["C++"]
date: 2020-08-13T15:19:43-04:00
lastmod: 2020-08-13T15:19:43-04:00
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





## Why?

Recently someone told me the IDE often suggests  `Clang-Tidy: Use emplace_back instead of push_back`, but he don't quite understand what is the difference between `emplace_back` and `push_back`. I happen to review the Scott Meyers's book "Effective Modern C++" few months ago, so I think it is a good time to summarise the pros and cons of `emplace_back`  method based on my experience.





## Efficiency

First of all, let's start from the definition of each method:



**push_back**: Adds a new element at the end of the container, after its current last element. The content of *val* is **copied (or moved)** to the new element.



**emplace_back**: Inserts a new element at the end of the container, right after its current last element. This new element is **constructed in place** using *args* as the arguments for its constructor.



To be more clear, what will happen if we call `push_back`?

1. A constructor will be called to create a temporary object.
2. A copy of the temporary object will be constructed in the memory for the container. Note that the `move constructor` will be called if exist because the temporary object is an `rvalue`, otherwise the `copy constructor` should be called. 
3. The destructor will be called to destroy the temporary object after copy.



As a comparison, `emplace_back` directly takes **constructor arguments** for objects to be inserted. In other words, the emplacement function **avoids constructing and destructing temporary objects**. It will be much more efficient if developer try to insert large amount of objects or if each object is time consuming to create/destroy.  





## Example



Let's create a simple class:

```cpp
class MyClass {
public:
    MyClass(int x, int y) : x_(x), y_(y) {
        std::cout << "Create class" << std::endl;
    }

    ~MyClass() {
        std::cout << "Destroy class" << std::endl;
    }

    // Copy Constructor
    MyClass(const MyClass& my_class) {
        std::cout << "Copy Constructor Called" << std::endl;
        x_ = my_class.x_;
    }

    // Move Constructor
    MyClass (MyClass&& my_class) noexcept {
        std::cout << "Move Constructor Called" << std::endl;
        x_ = std::move(my_class.x_);
    }

private:
    int x_ = 0;
    int y_ = 0;

};
```



In the code above, we print out some debugging statements in each constructor, so that we could have a better understanding of how the flow works.

 

> Refer to [here](https://github.com/facontidavide/CPP_Optimizations_Diary/blob/master/docs/reserve.md) for why we need to reserve space in advance.

```cpp
int main() {
    std::vector<MyClass> vector;
    // Reserve space to avoid reallocation
    vector.reserve(2);
    
    std::cout << "\n--- push_back ---" << std::endl;
    vector.push_back(MyClass(1, 2));
    
    std::cout << "\n--- emplace_back ---" << std::endl;
    vector.emplace_back(1, 2);
    
    std::cout << "\n--- Finish ---" << std::endl;
    return 0;
}
```



Output:

```bash
--- push_back ---
Create Class
Move Constructor Called
Destroy Class

--- emplace_back ---
Create Class

--- Finish ---
Destroy Class
Destroy Class
```



As we expected, `push_back` method calls the move constructor to make a copy and the destructor to destroy the temporary object. But `emplace_back` construct the object directly.



With the simple benchmark here, we notice that `emplace_back` is **7.62%** faster than `push_back` when we insert 1,000,000 object (MyClass) into an vector.

```
Insert 1,000,000 objects.

--- push_back ---
push_back takes 0.00665344 seconds.

--- emplace_back ---
emplace_back takes 0.00614631 seconds.
```





## Why not use `emplace_back` all the time?



We elaborate some of benefits of `emplace_back` method above, you might want to ask - Can we just get rid of the `push_back` method of containers?



Consider following code segment:

```cpp
vec1.push_back(1000000);
vec2.emplace_back(1000000);
```



For the first vector, we can tell it tries to add the number 1,000,000 to the end of the vector. However the behaviour of second line is not clear. Without knowing the type of the vector, we don’t know what constructor is actually invoking.



For example, if the vector is defined as:

```cpp
std::vector<std::vector<int>> vec1, vec2;
// vec1.push_back(1000000);  // compile error 
vec2.emplace_back(1000000);
std::cout << "Vector Size = " << vec2.at(0).size() << std::endl;
```



Here is the output:

```bash
Vector Size = 1000000
```



In the example above, it constructs a `vector` of 1,000,000 elements, allocating several megabytes of memory in the process. In most of the cases, it is unexpected and may take longer time for the developers to catch similar issues.



In some cases the transformation would be valid, but the code wouldn’t be exception safe. In this case the calls of `push_back` won’t be replaced.



```cpp
std::vector<std::unique_ptr<int>> v;
v.push_back(std::unique_ptr<int>(new int(0)))
```



This is because replacing it with `emplace_back` could cause a leak of this pointer if `emplace_back` would throw exception before emplacement (e.g. not enough memory to add a new element).



And there are some other corner cases and good examples that `push_back` outperforms than `emplace_back`, such as container with set of unique objects, or interact with `explicit` constructors. More details can be found in ''Item 42: Consider emplacement instead of insertion" in the book "Effective Modern C++".



At the end, I would love to quote the suggestions from this [blog](https://abseil.io/tips/112):

>  Very often the performance difference just won’t matter. As always, the rule of thumb is that you should avoid “optimizations” that make the code less safe or less clear, unless the performance benefit is big enough to show up in your application benchmarks.



## References

1. [CLANG-TIDY - MODERNIZE-USE-EMPLACE](https://clang.llvm.org/extra/clang-tidy/checks/modernize-use-emplace.html)
2. [C++ difference between emplace_back and push_back function](http://candcplusplus.com/c-difference-between-emplace_back-and-push_back-function)
3. [Tip of the Week #112: emplace vs. push_back](https://abseil.io/tips/112)

