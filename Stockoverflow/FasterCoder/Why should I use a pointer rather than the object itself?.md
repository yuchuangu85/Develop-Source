<h1 align="center">Why should I use a pointer rather than the object itself?</h1>

## question

I'm coming from a Java background and have started working with objects in C++. But one thing that occurred to me is that people often use pointers to objects rather than the objects themselves, for example this declaration:

```cpp
Object *myObject = new Object;
```

rather than:

```cpp
Object myObject;
```

Or instead of using a function, let's say `testFunc()`, like this:

```cpp
myObject.testFunc();
```

we have to write:

```cpp
myObject->testFunc();
```

But I can't figure out why should we do it this way. I would assume it has to do with efficiency and speed since we get direct access to the memory address. Am I right?

## answer

It's very unfortunate that you see dynamic allocation so often. That just shows how many bad C++ programmers there are.

In a sense, you have two questions bundled up into one. The first is when should we use dynamic allocation (using `new`)? The second is when should we use pointers?

The important take-home message is that you should **always use the appropriate tool for the job**. In almost all situations, there is something more appropriate and safer than performing manual dynamic allocation and/or using raw pointers.

### Dynamic allocation

In your question, you've demonstrated two ways of creating an object. The main difference is the storage duration of the object. When doing `Object myObject;` within a block, the object is created with automatic storage duration, which means it will be destroyed automatically when it goes out of scope. When you do `new Object()`, the object has dynamic storage duration, which means it stays alive until you explicitly `delete` it. You should only use dynamic storage duration when you need it. That is, **you should \*always\* prefer creating objects with automatic storage duration when you can**.

The main two situations in which you might require dynamic allocation:

1. **You need the object to outlive the current scope** - that specific object at that specific memory location, not a copy of it. If you're okay with copying/moving the object (most of the time you should be), you should prefer an automatic object.
2. **You need to allocate a lot of memory**, which may easily fill up the stack. It would be nice if we didn't have to concern ourselves with this (most of the time you shouldn't have to), as it's really outside the purview of C++, but unfortunately, we have to deal with the reality of the systems we're developing for.

When you do absolutely require dynamic allocation, you should encapsulate it in a smart pointer or some other type that performs [RAII](http://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization) (like the standard containers). Smart pointers provide ownership semantics of dynamically allocated objects. Take a look at [`std::unique_ptr`](http://en.cppreference.com/w/cpp/memory/unique_ptr) and [`std::shared_ptr`](http://en.cppreference.com/w/cpp/memory/shared_ptr), for example. If you use them appropriately, you can almost entirely avoid performing your own memory management (see the [Rule of Zero](https://en.cppreference.com/w/cpp/language/rule_of_three)).

### Pointers

However, there are other more general uses for raw pointers beyond dynamic allocation, but most have alternatives that you should prefer. As before, **always prefer the alternatives unless you really need pointers**.

1. **You need reference semantics**. Sometimes you want to pass an object using a pointer (regardless of how it was allocated) because you want the function to which you're passing it to have access that that specific object (not a copy of it). However, in most situations, you should prefer reference types to pointers, because this is specifically what they're designed for. Note this is not necessarily about extending the lifetime of the object beyond the current scope, as in situation 1 above. As before, if you're okay with passing a copy of the object, you don't need reference semantics.
2. **You need polymorphism**. You can only call functions polymorphically (that is, according to the dynamic type of an object) through a pointer or reference to the object. If that's the behavior you need, then you need to use pointers or references. Again, references should be preferred.
3. **You want to represent that an object is optional** by allowing a `nullptr` to be passed when the object is being omitted. If it's an argument, you should prefer to use default arguments or function overloads. Otherwise, you should preferably use a type that encapsulates this behavior, such as `std::optional` (introduced in C++17 - with earlier C++ standards, use `boost::optional`).
4. **You want to decouple compilation units to improve compilation time**. The useful property of a pointer is that you only require a forward declaration of the pointed-to type (to actually use the object, you'll need a definition). This allows you to decouple parts of your compilation process, which may significantly improve compilation time. See the [Pimpl idiom](http://en.wikipedia.org/wiki/Opaque_pointer).
5. **You need to interface with a C library** or a C-style library. At this point, you're forced to use raw pointers. The best thing you can do is make sure you only let your raw pointers loose at the last possible moment. You can get a raw pointer from a smart pointer, for example, by using its `get` member function. If a library performs some allocation for you which it expects you to deallocate via a handle, you can often wrap the handle up in a smart pointer with a custom deleter that will deallocate the object appropriately.

## source

[Why should I use a pointer rather than the object itself?](https://stackoverflow.com/questions/22146094/why-should-i-use-a-pointer-rather-than-the-object-itself)

