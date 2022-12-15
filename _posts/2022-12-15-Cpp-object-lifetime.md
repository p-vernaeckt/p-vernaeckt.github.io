---
title: C++ objects lifetime
date: 2022-12-15 13:51:00 +0200
categories: [C++]
tags: [c++, object lifetime]     # TAG names should always be lowercase
toc: true
---

Object lifetime in C++ refers to the time during which an object exists and is accessible in a program. Understanding object lifetime is important because it affects the behavior of a program and can impact the performance of the code. In this blog post, we will discuss object lifetime in C++ and the different types of constructors and destructors that are used to manage object lifetime.

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
