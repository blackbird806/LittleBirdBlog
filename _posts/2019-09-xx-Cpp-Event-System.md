---
layout: post
title: "An event system in C++"
date: 2019-09-xx 
comments: false
---

A nice feature of C# I like a lot is [multicast delegate](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-combine-delegates-multicast-delegates), which allow us to add multiple function in one delegate and then call them all from that delegate.

In C++ delegates are not part of the core language but can be substitued with function pointers or functors (like std::function), however multicast delegates cannot be easly substitued.

So let's see how we can implement a such feature in C++.

First of all we need to define how we want to use this feature.
Personnaly I like a lot the C# style so I'll try to get close to both it's syntax and it's behavior.

So we'll try to get something like this:

```c++
struct Foo
{
    int data = 0;
    void print(int add)
    {
        std::cout << data + add << "\n";
    }
};

void printNum(int num)
{
    std::cout << num << "\n";
}

int main()
{
    Event<void(int)> myEvent;
    
    myEvent += printNum;
    myEvent += printNum;

    Foo f;
    f.data = 12;

    myEvent += {&f, &Foo::print};
    myEvent += {&f, &Foo::print};
    myEvent += [](int i){ std::cout << i + 5 << "\n"; };

    myEvent(22);

    myEvent -= {&f, &Foo::print};
    myEvent -= printNum;
    std::cout << "\n";
    myEvent(11);

    return 0;
}
```
And output this :
```
22
22
34
34
27

16
```

The += operator is used to add function or method to the event and the -= operator is used to remove <u>all</u> similar functions to the event.

A first try to achieve this is to use a std::vector of std::function and then iterate and call all theses functions when the event is called.
Let's try this :

```c++
#include <iostream>
#include <utility>
#include <functional>
#include <vector>
#include <algorithm>

struct bar
{
    int test(double d, float f)
    {
        std::cout << d - f << "\n";
        return d - f;
    }
};

int foo(double d, float f)
{
    std::cout << d + f << "\n";
    return d + f;
}


// Here we use partial template specialisation to have the same syntax as std::function.
template<typename T>
struct Event;

template<typename R, typename ...ArgTypes>
struct Event<R (ArgTypes...)>
{
    void operator()(ArgTypes&&... args) const
    {
        for (auto const& fct : functions)
        {
            fct(std::forward<ArgTypes>(args)...);
        }
    }

    void operator+=(std::function<R (ArgTypes...)> fct)
    {
        functions.push_back(fct);
    }

    std::vector<std::function<R (ArgTypes...)>> functions;
};

int main()
{
    Event<int(double, float)> e;
    bar b;
    e += foo;
    e += foo;
    e += foo;
    e += [&b](double d, float f) -> int { return b.test(d, f); };
    e(5, 3);

    return 0;
}
```

which outputs :
```
8
8
8
2
```

This works great ! however you may noticed that I didn't implemented the -= operator. In fact unfortunatly we can't really compare two std::function, which is an operation needed to remove a specific function from the event (in fact there is a workaround using the [target](https://fr.cppreference.com/w/cpp/utility/functional/function/target) member function to compare the underlying function pointers, however this works only for non member functions).

So we do not have the choice here, we have to create our own function abstraction to create this event system.
So let's enumerate the features we need for this new abstraction :  
First we need our functions to be comparable between them in order to implement the -= operator. 
We also need to support both member and non member functions.
And finaly we want to support lambdas as well.

