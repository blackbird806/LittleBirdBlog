---
layout: post
title: "Polymorphic allocators in C++17"
date: 2019-11-16
comments: false
---

A nice and underrated feature of C++ 17 is polymorphic allocators.
As you may know most containers of the STL are defined by their contained type as well as their allocator type thanks to templates.  
So when we write :
```cpp
std::vector<int> arr;
```
We in fact have :
```cpp
std::vector<int, std::allocator<int>> arr;
```
However this design comes with some problems, for example the assignement of containers of different allocator types :

```cpp
std::vector<int> arr;
std::vector<int, custom_allocator<int>> other_arr;
arr = other_arr; // this do not compile since arr and other_arr are not of the same type
```

One way to handle this problem is to write an overload of '=' operator for theses types, however that's not a really viable solution because if you want to support other allocators, other containers and other operators that's lead to a lot of code you'll need to write (and I'm not even talking of copy/move construction problem).
So if overloading is not a viable solution.  
So how can we handle this problem properly ?

The answer is of course polymorphic allocators (I will shorten it to pma).  
The purpose of pma is to give the same allocator type : [std::pmr::polymorphic_allocator](https://en.cppreference.com/w/cpp/memory/polymorphic_allocator) to all your containers with custom allocation policy and then at runtime give them different memory resources to give them different allocations strategies.  
A [memory resource](https://en.cppreference.com/w/cpp/memory/memory_resource) is a class that will handle the memory allocation strategy asked by a pma.
So when you create a container with a pma you may want provide a memory resource to control allocation strategy. (If you do not provide one the STL will use a default one, which can be queried and modified with [get_default_resource](https://en.cppreference.com/w/cpp/memory/get_default_resource) and [set_default_resource](https://en.cppreference.com/w/cpp/memory/set_default_resource)).

You can create your own memory resources by inheriting from std::pmr::memory_resource, but for the example we will use some predefined memory resources of the STL.

So with pma we can write :
```cpp
std::pmr::monotonic_buffer_resource monotonic_res;
std::pmr::unsynchronized_pool_resource pool_res;

// arr and arr2 are of the same type but will allocate with different strategies because they are created with different memory_resource
std::vector<int, std::pmr::polymorphic_allocator<int>> arr(&monotonic_res);
std::vector<int, std::pmr::polymorphic_allocator<int>> arr2(&pool_res);

arr = arr2; // this is ok since arr and arr2 are of the same type
```

A [monotonic_buffer_resource](https://en.cppreference.com/w/cpp/memory/monotonic_buffer_resource) is a memory resource that allow very fast allocation but can only release it's memory all at once, and a [synchronized_pool_resource](https://en.cppreference.com/w/cpp/memory/synchronized_pool_resource) is a thread safe collection of pools.
I will not dig deeper than that on allocation strategies since it's not the topic here, butI may write an article on it later.

You may already noticed it but another cool thing about pma is that since the allocation strategy depend now on the memory resource rather than the allocator type, we can decide at runtime of the allocation strategy rather at compile time.
So we can do some stuff like this :

```cpp
std::unique_ptr<std::pmr::memory_resource> mem_res;

if (some_condition())
	mem_res = std::make_unique<std::pmr::monotonic_buffer_resource>();
else
	mem_res = std::make_unique<std::pmr::synchronized_pool_resource>();

// this is the same as std::vector<int, std::pmr::polymorphic_allocator<int>>
std::pmr::vector<int> arr(mem_res.get());

/* so now the memory resource of arr (so the allocation strategy) is either monotonic_buffer_resource or synchronized_pool_resource
depending of the value returned by some_condition() */
```

And that's it, if you wnat to go deeper with polymorphic allocators and memory resources I encourage you to read [the STL documentation](https://en.cppreference.com/w/cpp/memory) on it.