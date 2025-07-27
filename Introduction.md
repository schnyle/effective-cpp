# Notes

## Introduction

- Unless I have a good reason for allowing a constructor to be used for implicit type conversions, I declare it explicit. [page 5]

## Chapter 1

### Item 1: View C++ as a federation of languages

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

the symbolic name `ASPECT_RATIO`

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
