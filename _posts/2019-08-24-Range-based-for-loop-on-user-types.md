---
layout: post
title: "Range based for loop on user defined types"
date: 2019-08-24
comments: false
---

A great and very appreciated C++11 feature is Range based for loop which allow us to write loops with a cleaner syntax than before.

For example this code in C++03 :
```cpp
int array[] = {0, 1, 2, 3, 4, 5};
for (int i = 0; i < arraySize; i++)
    std::cout << array[i] << " ";
```
Can be converted in this in C++11 :

```cpp
int array[] = {0, 1, 2, 3, 4, 5};
for (auto const& it : array)
    std::cout << it << " ";
```

In this case the syntax modification is not huge but It's more intersting with more complex data structures like std::vector.

C++03 version :
```cpp
std::vector<int> array(6);
for (int i = 0; i < array.size(); i++)
    array[i] = i;
    
for (std::vector<int>::iterator it = array.begin(); it != array.end(); ++it)
{
    std::cout << *it << " ";
}
```
C++11 version :
```cpp
std::vector array = {0, 1, 2, 3, 4, 5};
for (auto const& it : array)
{
    std::cout << it << " ";
}
```

With always the same output :
```
0 1 2 3 4 5 
```
<!--more-->
Here the new syntax is a way more compact without be less readable, but things get really interesting when we look on associatives data structures like std::map or std::unordered_map, thanks to structured bindings of C++17.

So this code in C++03 : 
```cpp
std::map<std::string, int> m;
m["a"] = 1;
m["b"] = 2;
m["c"] = 3;
m["d"] = 4;
for (std::map<std::string, int>::iterator it = m.begin(); it != m.end(); ++it)
    std::cout << it->first << " : " << it->second << "\n";
```

Can be replaced with this in C++17 :
```cpp
std::map<std::string, int> m;
m["a"] = 1;
m["b"] = 2;
m["c"] = 3;
m["d"] = 4;
for (auto const& [key, value] : m)
    std::cout << key << " : " << value << "\n";
```
And the output is of course always the same :
```
a : 1
b : 2
c : 3
d : 4
```

Here we can see why makes this feature is so interesting.  
But if we can use it with STL containers why not use it with ours ?

First of all let's look what's behind the scene when using range based for loops.
According to the standard a for range loop produce this code in C++17/20 (it's almost the same before C++17/20 however. This modification is due to that : since C++17 begin_expr and end_expr can be different types as long as they can be compared) :
> ```cpp
> {
>   init-statement //this line is in C++20 only
>   auto && __range = range_expression ;
>   auto __begin = begin_expr ;
>   auto __end = end_expr ;
>   for ( ; __begin != __end; ++__begin) 
>   {
>       range_declaration = *__begin;
>       loop_statement
>   }
>} 
> ```

And always according to the standard :
> If range_expression is an expression of a class type C that has both a member named begin and a member named end (regardless of the type or accessibility of such member), then begin_expr is __range.begin() and end_expr is __range.end();

from : [https://en.cppreference.com/w/cpp/language/range-for](https://en.cppreference.com/w/cpp/language/range-for)  

So now we have all what we need to use range based for loops with our own classes. As we saw it above we first need a class that implement the method begin() which return an iterator that must be comparable with the value returned by end(), must implement the pre increment operator and must be dereferenceable.
So let's start with a very simple example in C++17:
```cpp
#include <iostream>

class CustomArray
{
    int data[100];

    public:
    CustomArray()
    {  
        for (int i = 0; i < 100; i++)
            data[i] = i;
    }

    auto begin() const noexcept
    {
        return data;
    }

    auto end() const noexcept
    {
        return &data[100];
    }
};


int main()
{
    CustomArray c;
    for (auto e : c)
        std::cout << e << " ";

    return 0;
}
```

which outputs :
```
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 ...
```

Here there is no much to explain, begin() return a pointer to the first element of our array and end() to the last element, since a pointer fulfilled the conditions described above the code compile and produce the desired output.

But let's see what's behind with [CppInsight](https://cppinsights.io/s/30498505).
```cpp
CustomArray c = CustomArray();
{
  CustomArray & __range1 = c;
  const int * __begin1 = __range1.begin();
  const int * __end1 = __range1.end();
  for(; __begin1 != __end1; ++__begin1) 
  {
    int e = *__begin1;
    std::operator<<(std::cout.operator<<(e), " ");
  }
}
```

This is the code "produced" by our range based for loop. So here we can see what the specification described to us before.
__range1 is a reference on our container, __begin1 contain our iterator, in our case a int* , __end1 contain an element comparable with __begin1 which is in our case also an int* and then we can see that the element 'e' contain the return value of the iterator dereferencement.
Then we enter in our loop body and print the element.

Now let's look to a more complex example : 

```cpp
#include <iostream>
#include <map>
#include <tuple>

template<typename K, typename V, size_t Size>
struct myMap
{
    K keys[Size];
    V values[Size];

    struct iterator
    {
        myMap& ref;
        size_t index;

        auto operator*()
        {
            return std::make_tuple(index, ref.keys[index], ref.values[index]);
        }

        iterator& operator++()
        {
            index++;
            return *this;
        }

        bool operator!=(iterator const& other)
        {
            return index != other.index;
        }
    };

    auto begin() noexcept
    {
        return iterator{*this, 0};
    }

    auto end() noexcept
    {
        return iterator{*this, Size};
    }
};

int main()
{
    myMap<float, char, 3> m;
    m.keys[0] = 1.2;
    m.values[0] = 'a';
    m.keys[1] = 2.3;
    m.values[1] = 'b';    
    m.keys[2] = 3.4;
    m.values[2] = 'c';

    for (auto&& [index, key, value] : m)
        std::cout << index << " : " << key << " : " << value << "\n";

	return 0;
}
```
which outputs us : 
```
0 : 1.2 : a
1 : 2.3 : b
2 : 3.4 : c
```

In this example I used my own iterators instead of pointers, so I have to implement all needed functions in order to use range based for loops.
On top of that I decided to return a tuple with the dereferencement operator in order to use structured bindings in my loop.
And now we can see the magic happen, a beautiful for loop of 2 lines which iterate on all my container.  

And that's it, of course we've only scratched the surface of all the possibilities that offer this feature however now you should understand it; and be able to explore all theses possibilities by yourself.