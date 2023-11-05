---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/
# image: https://cooltext.com/

title: "C++ Diary #2 | std::vector::reserve"
subtitle: ""
summary: "Using std::vector::reserve whenever possible"
authors:
- admin
tags:
- cpp
categories: ["C++"]
date: 2020-12-26T19:53:55-05:00
lastmod: 2020-12-26T19:53:55-05:00
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



## How a `std::vector` Grows



In C++, std::vector provides the capability of creating dynamic arrays. In order to support random element access, the elements in vector container will be assigned to contiguously memory blocks. Considering we pre-allocate certain amount of memory and vector can grow (or shrink) dynamically, here we might raise a question:



> When we add a new element, what will happen if there is no space left for the new element?



Obviously, the vector container have to do following steps:

1. Allocate a new and larger contiguous memory space
2. Move existing elements into the new memory location
3. Add current new element to memory
4. Deallocate the old memory space



> Do we need to allocation and deallocation each time we add an new element?



We cannot avoid performance degradation if memory allocation and deallocation happens every single time. So what is the mechanism to optimize it in  standard library? Here we start with an example.



First of all, we create a simple class with some print statements for debugging purpose:

```cpp
class MyClass {
public:
    explicit MyClass(int x) : x_(x){
        std::cout << "Create Class with number " << x_ << std::endl;
    }

    ~MyClass() {
        std::cout << "Destroy Class with number " << x_ << std::endl;
    }
    
    // Move Constructor
    MyClass (MyClass&& my_class) noexcept {
        std::cout << "Moving " << my_class.x_ << std::endl;
        x_ = std::move(my_class.x_);
    }

private:
    int x_ = 0;
};
```



Here we create a simple task to add numbers into an vector, and we also print out the [capacity](http://www.cplusplus.com/reference/vector/vector/capacity/)(currently allocated storage) of the vector:

```cpp
int main() {
    std::vector<MyClass> v;
    std::cout << "\n--- Start ---" << std::endl;

    constexpr int n = 9;
    for (int i = 0; i < n; i++) {
        std::cout << "Insert " << i << std::endl;
        std::cout << "Current Capacity = " << v.capacity() << std::endl;
        v.emplace_back(i);
        std::cout << "New Capacity = " << v.capacity() << std::endl;
        std::cout << std::endl;
    }

    std::cout << "\n--- Finish ---" << std::endl;
    return 0;
}
```



And let's take a look at the result:

```bash
Insert 0
Current Capacity = 0
Create Class with number 0
New Capacity = 1

Insert 1
Current Capacity = 1
Create Class with number 1
Moving 0
Destroy Class with number 0
New Capacity = 2

Insert 2
Current Capacity = 2
Create Class with number 2
Moving 0
Moving 1
Destroy Class with number 0
Destroy Class with number 1
New Capacity = 4

Insert 3
Current Capacity = 4
Create Class with number 3
New Capacity = 4

...

Insert 7
Current Capacity = 8
Create Class with number 7
New Capacity = 8

Insert 8
Current Capacity = 8
Create Class with number 8
Moving 0
Moving 1
Moving 2
Moving 3
Moving 4
Moving 5
Moving 6
Moving 7
Destroy Class with number 0
Destroy Class with number 1
Destroy Class with number 2
Destroy Class with number 3
Destroy Class with number 4
Destroy Class with number 5
Destroy Class with number 6
Destroy Class with number 7
New Capacity = 16
```



We notice that the capacity increases from 0->1->2->4->8->16, it indicates that the `std::vector` actually double the capacity each time when it have to allocate larger memory. In other words, it will allocate more memory that is beyond the immediately needs. For example, it allocate new capacity of 16 when we only store 8 elements in the vector.



However, we should also catch that the class's move (or copy) constructor and the destructor is called multiple times, in order to move elements into new memory location. If we have a relatively complex class with lots of member variables to copy and destroy, then overall time may be **non-negligible**.



## std::vector::reserve



A call to `vector::reserve(n)`  requests the amount of memory which is at least enough to carry $n$ elements:

```cpp
// Reserve space to avoid reallocation
v.reserve(500);
```



As a result, it is not necessary to reallocate memory every time, because we already reserved enough amount of memory ahead.

```bash
Insert 0
Current Capacity = 500
Create Class with number 0
New Capacity = 500

Insert 1
Current Capacity = 500
Create Class with number 1
New Capacity = 500

Insert 2
Current Capacity = 500
Create Class with number 2
New Capacity = 500

...

Insert 8
Current Capacity = 500
Create Class with number 8
New Capacity = 500
```



## reserve vs. resize



Vector container have two kinds of capacity parameters, `size` and `capacity`:

1. `vector::size` returns the number of actual object held in the vector, which is not necessarily equal to the storage capacity. We could use `vector::resize` to change the number of real elements by inserting or erasing elements from it. It will change `capacity` only when the resize number is greater than the current container's capacity.
2. `vector::capacity` return the size of the storage space currently allocated for the vector. It can be equal or greater than current number of elements in the vector. `vector::reserve` can request a change in `capacity`, however it does not effect the `size` of vector or its actual elements.



## Summary 



If we have a rough estimation of the number of elements we will store into the vector, we should then use`std::vector::reserve` whenever possible to avoid frequently memory reallocation.





## References

1. C++ Primer (5th Edition): Chapter 9.4
2. [Using std::vector::reserve whenever possible](https://www.geeksforgeeks.org/using-stdvectorreserve-whenever-possible/)
3. [cplusplus.com - std::vector::reserve](http://www.cplusplus.com/reference/vector/vector/reserve/)