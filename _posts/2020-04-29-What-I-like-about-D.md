---
layout: post
title: "What I like about D"
date: 2020-04-29
comments: false
---

```D
import std.stdio;

void main() {
	writeln("Hello world");
}
```
> A hello world in Dlang.

Some times ago I discovered [the D programming language](https://dlang.org/) which is now one of my favourite if not my favourite language.
So in this post I 'll show you some features that I love in D.

But first of all, a little intro.

# The D programming language 

D was created in 2001 by Walter Bright with lessons learned from C++. D is really close to C++ (especially modern C++) but have much more features, a cleaner syntax, faster compile times, and so on.
D have also a fully optional garbage collector to handle memory (this mean you can disable it in some part of your programs or even disable it in your whole project), which allow to write more compact and readable code.


# Some Great D features

For the following of this post I will assume you already know most of C++ features in order to compare it to D.
I will also not show [every D feature](https://dlang.org/spec/spec.html) since there is a way too much.

### D templates

D's syntax for templates is a little shorter than C++ :
```D
import std.stdio;
// here T is the type name
auto add(T)(T a, T b) {
	return a + b;
}

void main() {
	writeln("a + b = ", add(5, 3));
}
```
But where it become interesting is when we use template in a little more advanced way :

```D
import std.stdio;

void printEven(ArgTypes...)(ArgTypes args) {
	foreach (i, arg; args) {
		if (i % 2 == 0) write(arg, " ");
	}
}

void main() {
	printEven(0, "hello", "world", 5.4f);
}
```
output : 
> ``` 0 world ```

The following code iterate on each argument of the variadic template with the foreach statement and then print only the even arguments. Yes it's that easy to iterate on variadic template parameters in D.
That's what I love with D, metaprogramming and template stuff is a way easier, more compact and more powerfull than in C++.
Let's look to another example :
```D
import std.stdio;

void templatePrint(string str)() {
	pragma(msg, str);
}

void main() {
	templatePrint!("hello")();
}
```
D support NTTP (Non Type Template Parameter), a feature comming in C++20, but which D support since a long time ago.
The above exmaple will print hello during compilation, since str is a template parameter, the ```pragma(msg, str);``` is a special statement that told the compiler to print ``` str ``` during compilation.

### CTFE 

D have also a really powerfull feature called CTFE (Compile Time Function Evaluation), there is a similar feature in modern C++ with constexpr and consteval.
A simple example:
```D
import std.stdio;

int add(int a, int b) {
	return a + b;
}

pragma(msg, add(5, 2));

void main() {
	writeln(add(4, 1));
}
```

This code will print 7 at compile time and 5 at run time, the same function can be used both at compile time and at runtime without any specifiers unlike C++.

### The true power of D metaprogramming

I see you, saying "yeah theses features are great, but C++ have them too".
Now I'll show you why D metaprogramming is so powerfull and why C++ is a way behind D in this domain.

String mixins.  
With string mixin D can compiles an arbitrary string as D code __At compile time__.

```D
mixin("int b = 5;");
assert(b == 5); // compiles just fine
```
> Okay but what's the use of a such feature ?

Remember NTTP ?
Let's look at this example from Dlang.org.

```D
import std.stdio : writeln;

auto calculate(string op, T)(T lhs, T rhs) {
    return mixin("lhs " ~ op ~ " rhs");
}

void main() {
    // pass the operation to perform as a
    // template parameter.
    writeln("5 + 12 = ", calculate!"+"(5,12));
    writeln("10 - 8 = ", calculate!"-"(10,8));
    writeln("8 * 8 = ", calculate!"*"(8,8));
    writeln("100 / 5 = ", calculate!"/"(100,5));
}
```

Now you see it right ?

We can even go further with D's static reflection, and template mixins:

```D
import std.stdio : writeln;

struct Vec3 {
    float x, y, z;
}

string getMemberArrays(T)()
if (is(T == struct)) {// template constraint T must be a struct
    string memArray;
    foreach (mem; __traits(allMembers, Vec3)) {
        // array / string concatenation can be done with ~ operator
        memArray ~= typeof(mixin("T." ~ mem)).stringof // get the type name of the current member
        ~ "[] "         // make an array of this type 
        ~ mem ~";\n";   // keep the same name as the original struct
    }
    return memArray;
}

// mixin template are used to generate boilerplate code
// it works like a C++ macro but at the AST level, so we can use template parameters with it.
mixin template makeSOA(T) {
    // declare the new struct with a prefix
    mixin("struct SOA_" ~ T.stringof ~ "{ \n" 
    ~ getMemberArrays!T // add the members as arrays
    ~ "}");
}

mixin makeSOA!Vec3; // generate the struct

void main() {
    SOA_Vec3 v;
    v.x = [5, 3, 2, 1];
    v.y = [2, 3, 1, 5];
    v.z = [7, 2 ,10 ,1];
    writeln(v);
}
```

Here we have a simple metaprogram that take a struct as input and will generate as output another struct with an array of each of his members.
And of course all of this is generated at compile time. 
There is some features wee didn't saw in the code above, but the goal here is more to understand the power and the possibilities of D metaprogramming rather than a tutorial (I may write one in the future though).


# Going further

I highly encourage you to try [D](https://dlang.org/), do the [Dlang tour](https://tour.dlang.org/) (a really great tutorial about the D basics).
On my side I will certainly write more about D, so stay tuned.