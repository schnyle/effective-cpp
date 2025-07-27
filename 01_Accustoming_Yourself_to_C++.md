# Chapter 1: Accustoming Yourself to C++

## Item 1: View C++ as a federation of languages

To make sense of C++, you have to recognize its primary sub-languages. Fortunately, there are only four:

- **C.** In many cases, C++ offers approaches to problems that are superior to their C counterparts, but when you find yourself working with the C part of C++, the rules for effective programming reflect C's more limited scope: no templates, no exceptions, no overloading, etc.
- **Object-Oriented C++.** This is the part of C++ to which the classic rules for object-oriented design most directly apply.
- **Template C++.** This is the generic programming part of C++, the one that most programmers have the least experience with.
- **The STL.** The STL has particular ways of doing things, and when you're working with the STL, you need to be sure to follow its conventions.

**Things to Remember**

- Rules for effective C++ programming vary, depending on the part of C++ you are using.

## Item 2: Prefer `const`s, `enum`s, and `inline`s to `#define`s.

This Item might better be called "prefer the compiler to the preprocessor", because `#define` may be treated as if it's not part of the language _per se_. `#define` can be problematic. For example:

```c++
#define ASPECT_RATIO 1.653
```

the symbolic name `ASPECT_RATIO` may never be seen by compilers; it may be removed by the preprocessor before the source code ever gets to a compiler. As a result, the name `ASPECT_RATIO` may not get entered into the symbol table. This can be confusing if you get an error during compilation involving the use of a constant, because the error message may refer to 1.653, not `ASPECT_RATIO`.

When replacing `#define`s with constants, two special cases are worth mentioning

1. Defining constant pointers. Because constant definitions are typically put in header files, it's important that the _pointer_ be declared `const`, usually in addition to what the pointer points to:

```c++
const char* const authorName = "Scott Meyers";
```

Reminder: `string` objects are generally preferable to their `char*`-based progenitors:

```c++
const std::string authorName("Scott Meyers");
```

2. Class-specific constants.

To limit the scope of a constant to a class, you must make it a member, and to ensure there's at most one copy of the constant, you must make it a _static_ member:

```c++
class GamePlayer {
  private:
    static const int NumTurns = 5;  // constant declaration
    int scores[NumTurns];           // use of constant
    ...
};
```

Older compilers may not accept the syntax above, because it used to be illegal to provide an initial value for a static class member at its point of declaration. Furthermore, in-class initialization is allowed only for integral types and only for constants. See page 15 for more on this exception and also "the enum hack" for when the constant is needed at class compile time.

Back to the preprocessor and macros, consider the following macro that calls some function `f` with the greater of the macro's arguments:

```c++
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

Whenever you write this kind of macro, you have to remember to parenthesize all the arguments in the macro body. Otherwise you can run into trouble when somebody calls the macro with an expression. Additionally:

```c++
int a = 5, b = 0;

CALL_WITH_MAX(++a, b);       // a is incremented twice
CALL_WITH_MAX(++a, b + 10);  // a is incremented once
```

The number of times that `a` is incremented before calling `f` depends on what it is being compared with!

Fortunately, you can get all the efficiency of a macro plus all the predictable behavior and type safety of a regular function by using a template for an inline function:

```c++
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
  f(a > b ? a : b);
}
```

**Things to Remember**

- For simple constants, prefer `const` objects or `enum`s to `#define`s.
- For function-like macros, prefer `inline` functions to `#define`s.

## Item 3: Use `const` whenever possible.

The wonderful thing about `const` is that it allows you to specify a semantic constraint - a particular object should _not_ be modified - and compilers will enforce that constraint. It allows you to communicate to both compilers and other programmers that a value should remain invariant. Whenever that is true, you should be sure to say so, because that way you enlist your compilers' aid in making sure the constraint isn't violated.

Having a function return a constant value often makes it possible to reduce the incidence of client errors without giving up safety or efficiency. For example, the declaration

```c++
class Rational { ... };
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

notifies the programmer of the following common typo:

```c++
Rational a, b, c;
...
if (a * b = c) ...
```

### `const` Member Functions

There are two prevailing notions of `const`: _bitwise constness_ or _physical constness_ and _logical constness_. Bitwise `const` says that a member function is `const` if and only if it doesn't modify any of the object's data members, i.e., if it doesn't modify and of the bits inside the object. Unfortunately, many member functions that don't act very `const` pass the bitwise test. In particular, a member function that modifies what a pointer _points_ to frequently doesn't act `const`, but since it is not modifying the _pointer_ itself, the function is bitwise `const`, and compilers won't complain. Logical constness says that a `const` member function might modify some of the bits in the object on which it's invoked, but only in ways that clients cannot detect. C++ has const-related wiggle room known as `mutable`:

```c++
class CTextBlock {
public:
  ...
  std::size_t length() const;
private:
  char* pText;
  mutable std::size_t textLength;  // these data members may
  mutable bool lengthIsValid;      // always be modified, even in
};                                 // const member functions

std::size_t CTextBlock::length() const
{
  if (!lengthIsValid) {
    textLength = std::strlen(pText);  // fine
    lengthIdValid = true;             // also fine
  }
  return textLength;
};
```

### Avoiding Duplication in `const` and Non-`const` Member Functions

```c++
class TextBlock {
public:
  ...
  const char& operator[](std::size_t position) const
  {
    ...
    ...
    ...
    return text[position];
  }

  char& operator[](std::size_t position)
  {
    return const_cast<char&>(                         // cast away const on op[]'s return type
      static_cast<const TextBlock&>(*this)[position]  // get the previously defined op[] by casting *this to const
    );
  }
};
```

**Things to Remember**

- Declaring something `const` helps compilers detect usage errors. `const` can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
- Compilers enforce bitwise constness, but you should program using conceptual constness.
- When `const` and non-`const` member functions have essentially the same identical implementations, code duplication can be avoided by having the non-`const` version call the `const` version.

## Item 4: Make sure that objects are initialized before they're used.

There are rules that describe when object initialization is guaranteed to take place and when it isn't. Unfortunately, the rules are complicated. In general, if you're in the C part of C++ and initialization would probably incur a runtime cost, it's not guaranteed to take place. If you cross into the non-C parts of C++, things sometimes change. This explain why a C array isn't necessarily guaranteed to have its contents initialized, but an STL `std::vector` is.

The best way to deal with this seemingly indeterminate state of affairs is to _always_ initialize your objects before you use them.

For non-member objects of built-in types, you'll need to do this manually.

For classes, use member initialization instead of assigning in the body of the constructor:

```c++
class PhoneNumber;
class ABEntry { ... };

ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
  : theName(name), theAddress(address), thePhones(phones), numTimesConsulted(0)
  {}
```

If this were assignment based would first call the default constructors to initialize `theName`, `theAddress`, and the `thePhones`, and then promptly assigned new values on top of the default-constructed ones. For objects of built-in type like `numTimesConsulted`, there is no difference in cost between initialization and assignment.

Once you've taken care of explicitly initializing non-member objects of built-in types and you've ensured that your constructors initialize their base classes and data members using the member initialization list, the only other thing to worry about is the order of initialization of non-local static objects defined in different translation units. Unfortunately, the relative order of initialization of non-local static objects defined in different translation units is undefined. Determining the "proper" order is a very hard problem. Fortunately, a small design change eliminates the problem entirely. All that has to be done is to move each non-local static object into its own function, where it's declared `static`. These functions return references to the objects they contain. Clients then call the functions instead of referring to the objects. In other words, non-local static objects are replaced with _local_ static objects. This is as a common implementation of the Singleton pattern.

**Things to Remember**

- Manually initialize objects of built-in type, because C++ only sometimes initializes them itself.
- In a constructor, prefer use of the member initialization list to assignment inside the body of the constructor. List data members in the initialization list in the same order they're declared in the class.
- Avoid initialization order problems across translation units by replacing non-local static objects with local static objects.
