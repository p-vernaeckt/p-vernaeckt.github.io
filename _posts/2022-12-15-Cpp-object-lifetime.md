---
title: C++ objects lifetime
date: 2022-12-15 13:51:00 +0200
categories: [C++]
tags: [c++, object lifetime]     # TAG names should always be lowercase
toc: true
---

Object lifetime in C++ refers to the time during which an object exists and is accessible in a program. Understanding object lifetime is important because it affects the behavior of a program and can impact the performance of the code. In this blog post, we will discuss object lifetime in C++ and the different types of constructors and destructors that are used to manage object lifetime.

Disclaimer: in what follows I will use [{fmt}](https://github.com/fmtlib/fmt) instead of `iostream`. I encourage you to do the same.

# Object Lifetime in C++
In C++, objects are created when they are instantiated using the new keyword or by declaring a variable with a class type. For example:

```c++
// Create an object using the new keyword
ExampleClass* obj1 = new ExampleClass();

// Create an object by declaring a variable
ExampleClass obj2;
```

The lifetime of an object begins when it is created and ends when it is destroyed. The lifetime of an object can be controlled using constructors and destructors.

# Constructors in C++
A constructor is a special member function of a class that is called automatically when an object of that class is created. Constructors are used to initialize the member variables of an object and to perform other tasks that are needed to set up the object for use.

There are several different types of constructors in C++, including the default constructor, the copy constructor, and the move constructor.

## Default Constructor
A default constructor is a constructor that takes no arguments and is called when an object is created without specifying any arguments. For example:

```c++
class ExampleClass {
public:
  // Default constructor
  ExampleClass() {
    // Initialize member variables here
  }
};

// Call default constructor
ExampleClass obj1;
```

## Copy Constructor
A copy constructor is a constructor that takes a reference to an object of the same class as the constructor and is called when an object is created by copying another object. For example:

```c++
class ExampleClass {
public:
  // Copy constructor
  ExampleClass(const ExampleClass& other) {
    // Copy member variables from other object here
  }
};

// Call default constructor
ExampleClass obj1;

// Call copy constructor
ExampleClass obj2 = obj1;
```

## Move Constructor
A move constructor is a constructor that takes a reference to an object of the same class as the constructor and is called when an object is created by moving another object. For example:

```c++
class ExampleClass {
public:
  // Move constructor
  ExampleClass(ExampleClass&& other) {
    // Move member variables from other object here
  }
};

// Call default constructor
ExampleClass obj1;

// Call move constructor
ExampleClass obj2 = std::move(obj1);

```

In this case, the guts of `obj1` are transfered to `obj2`, leaving `obj1` as an empty shell. Let's illustrate with a simple example involving an `std::string`:

```c++
#include <fmt/core.h>
#include <string>

int main() {
    // Create a string
    std::string s1{"Foo"};
    fmt::print("s1 = {}\n", s1);

    // Move it to another string
    std::string s2{std::move(s1)};
    fmt::print("s1 = {}\n", s1);
    fmt::print("s2 = {}\n", s2);

    return 0;
}
```

When executed, the following code produces this output:

```
s1 = Foo
s1 =
s2 = Foo
```

We can see that after `s1` has been moved using `std::move`, its content have been moved to s2. `s1` is thus an useless shell now.

To get a better understanding of how move can be used to improve performance, consider reading [this post]({% post_url 2022-12-15-move-semantics %}).

## Destructors in C++
A destructor is a special member function of a class that is called automatically when an object of that class is destroyed. Destructors are used to perform clean-up tasks, such as releasing memory that was allocated by the object.

```c++
class ExampleClass {
public:
  // Destructor
  ~ExampleClass() {
    // Perform clean-up tasks here
  }
};
```

## Object lifetime class
Ok now let's summarize everything, give our class a better name and a property. We will use this property to track our instances.

```c++
class LifetimeObserver {
private:
    uint8_t a;
public:
    LifetimeObserver():a{0} {
        fmt::print("No parameter constructor (a={})\n", a);
    }

    LifetimeObserver(uint8_t parameter):a{parameter} {
        fmt::print("Parameter constructor (a={})\n", a);
    }

    LifetimeObserver([[maybe_unused]] const LifetimeObserver& lto) {
        fmt::print("Copy constructor (a={})\n", a);
    }

    LifetimeObserver([[maybe_unused]] const LifetimeObserver&& lto) {
        fmt::print("Move constructor (a={})\n", a);
    }

    ~LifetimeObserver() {
        fmt::print("Destructor (a={})\n", a);
    }
};
```

The `[[maybe_unused]]` attribute silences compiling errors complaining that the parameter is not used in the function's body.

# Fooling around

Here is a complete example to make a good use of the `LifetimeObserver` class:

```c++
#include <fmt/core.h>
#include <string>
#include <memory>

class LifetimeObserver {
private:
    uint8_t a;
public:
    LifetimeObserver():a{0} {
        fmt::print("No parameter constructor (a={})\n", a);
    }

    LifetimeObserver(uint8_t parameter):a{parameter} {
        fmt::print("Parameter constructor (a={})\n", a);
    }

    LifetimeObserver([[maybe_unused]] const LifetimeObserver& lto):a{lto.a} {
        fmt::print("Copy constructor (a={})\n", a);
    }

    LifetimeObserver([[maybe_unused]] const LifetimeObserver&& lto):a{lto.a} {
        fmt::print("Move constructor (a={})\n", a);
    }

    ~LifetimeObserver() {
        fmt::print("Destructor (a={})\n", a);
    }
};


auto main() -> int {
    LifetimeObserver lto;
    LifetimeObserver lto1(99);
    LifetimeObserver lto2{lto1};
    LifetimeObserver lto3{std::move(lto2)};

    return 0;
}
```

When executed, this code produces this output:

```
No parameter constructor (a=0)
Parameter constructor (a=99)
Copy constructor (a=99)
Move constructor (a=99)
Destructor (a=99)
Destructor (a=99)
Destructor (a=99)
Destructor (a=0)
```

We can see here how every constructor can be called to achieve our needs.