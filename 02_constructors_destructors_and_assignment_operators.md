# Chapter 2: Constructors, Destructors, and Assignment Operators

## Item 5: Know what functions C++ silently writes and calls.

When is an empty class not an empty class? When C++ gets through with it. If you don't declare them yourself, compilers will declare their own versions of a copy constructor, a copy assignment operator, and a destructor. Furthermore, if you declare no constructors at all, compilers will also declare a default constructor for you. All these functions will be both
`public` and `inline`.

```c++
template<typename T>
class NamedObject {
public:
  NamedObject(const char *name, const T& value);
  NamedObject(const std::string& name, const T& value);
  ...
private:
  std::string nameValue;
  T objectValue;
};
```

Because a constructor is declared in `NamedObject`, compilers won't generate a default constructor. This is important. It means that if you've carefully engineered a class to require constructor arguments, you don't have to worry about compilers overriding your decision blithely adding a constructor that takes no arguments.

**Things to Remember**

- Compilers may implicitly generate a class's default constructor, copy constructor, copy assignment operator, and destructor.

## Item 6: Explicitly disallow the user of compiler-generated functions you do not want.

You might want to disallow copying for your class. In this case, if you don't declare a copy constructor or a copy assignment operator, compilers may generate them for you. The key to the solution is that all the compiler generated functions are public. To prevent these functions from being generated, you must declare them yourself, but there is nothing that requires _you_ declare them public. Instead, declare the copy constructor and the copy assignment operator _private_. By declaring a member function explicitly, you prevent compilers from generating their own version, and by making the function `private`, you keep people from calling it. If you are clever enough not to _define_ them, then attempting to use them produces and error at link time.

**Things to Remember**

- To disallow functionality automatically provided by compilers, declare the corresponding member functions `private` and give no implementations. Using a base class like `Uncopyable` is one way to do this.

## Item 7: Declare destructors virtual in polymorphic base classes.

C++ specifies that when a derived class object is deleted through a pointer to a base class with a non-virtual destructor, results are undefined. This is a recipe for disaster. What typically happens at runtime is that the derived part of the object is never destroyed. The base class part of the object typically would be destroyed, thus leading to a curious "partially" destroyed object.

Eliminating the problem is simple: give the base class a virtual destructor. Then deleting a derived class object will do exactly what you want. It will destroy the entire object, including all its derived class parts.

If a class does _not_ contain virtual functions, that often indicated it is not meant to be used as a base class. When a class is not intended to be a base class, making the destructor virtual is usually a bad idea. It adds extra overhead from the vtable and can break portability. Many people summarize the situation this way: declare a virtual destructor in a class if and only if that class contains at least one virtual function.

**Things to Remember**

- Polymorphic base classes should declare virtual destructors. If a class has any virtual functions, it should have a virtual destructor.
- Classes not designed to be base classes or not designed to be used polymorphically should not declare virtual destructors.

## Item 8: Prevent exceptions from leaving destructors.

**Things to Remember**

- Destructors should never emit exceptions. If functions called in a destructor may throw, the destructor should catch any exceptions, then swallow them or terminate the program.
- If class clients need to be able to react to exceptions thrown during an operation, the class should provide a regular (i.e. non-destructor) function that performs the operation.

## Item 9: Never call virtual functions during construction or destruction.

You shouldn't call virtual functions during construction or destruction, because they won't do what you think. Even if they did, you'd still be unhappy.

During base class construction, virtual functions never go down into derived classes. Instead, the object behaves as if it were of the base type. Informally speaking, during base class construction, virtual functions aren't.

**Don't call virtual functions during construction or destruction, because such calls will never go to a more derived class than that of the currently executing constructor or destructor.**

## Item 10: Have assignment operators return a reference to `*this`.

**Things to Remember**

- Have assignment operators return a reference to `*this`.

## Handle assignment to self in `operator=`.

Suppose you create a class that holds a raw pointer to a dynamically allocated bitmap:

```c++
class Bitmap {...};
class Widget {
  ...
private:
  Bitmap *pb;
};
```

Here's an implementation of `operator=` that looks reasonable on the surface but is unsafe in the presence of assignment to self.

```c++
Widget& Widget::operator=(const Widget& rhs)
{
  delete pb;
  pb = new Bitmap(*rhs.pb);
  return *this;
}
```

The self-assignment problem here is that inside `operator=`, `*this`and `rhs` could be the same object. When they are, the `delete` not only destroys the bitmap for the current object, it destroys the bitmap for `rhs`too. At the end of the function, the `Widget` - which should not have been changed by the assignment to self - finds itself holding a pointer to a deleted object! The traditional way to prevent this error is to check for assignment to self via an _identity test_ at the top of `operator=`:

```c++
Widget& Widget::operator=(const Widget& rhs)
{
  if (this == &rhs) return *this;

  delete pb;
  pb = new Bitmap(*rhs.pb);
  return *this;
}
```

This works, but it is also exception unsafe. In particular, if the "`new Bitmap`" expression yields an exception, the `Widget` will end up holding a pointer to a deleted `Bitmap`. Happily, making `operator=` exception-safe typically renders it self-assignment safe, too. As a result, it's increasingly common to deal with issues of self-assignment by ignoring them, focusing instead on achieving exception safety:

```c++
Widget& Widget::operator=(const Widget& rhs)
{
  Bitmap *pOrig = pb;
  pb = new Bitmap(*rhs.pb);
  delete pOrig;
  return *this;
}
```

**Things to Remember**

- Make sure `operator=` is well-behaved when an object is assigned to itself. Techniques include comparing addresses of source and target objects, careful statement ordering, and copy-and-swap.
- Make sure that any function operating on more than one object behaves correctly if two or more of the objects are the same.

## Item 12: Copy all parts of an object.

Any time you take it upon yourself to write copying functions for a derived class, you must take care to also copy the base class parts. Those parts are typically private, of course, so you can't just access them directly. Instead, derived class copying functions must invoke their corresponding base class functions:

```c++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs) : Customer(rhs), priority(rhs.priority)
{
}

PriorityCustomer& PriorityCusotmer::operator=(const PriorityCustomer& rhs)
{
  Customer::operator=(rhs);
  priority = rhs.priority;
  return *this;
}
```

The meaning of "copy all parts" in this Item's title should now be clear. When you're writing a copying function, be sure to (1) copy all local data members and (2) invoke the appropriate copying function in all base classes, too.

**Things to Remember**

- Copying functions should be sure to copy all of an object's data members and all of it's base class parts.
- Don't try to implement one of the copying functions in terms of the other. Instead, put common functionality in a third function that both call.
