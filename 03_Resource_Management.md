## Item 13: Use objects to manage resources.

To make sure that an acquired resource is always released, we need to put that resource inside an object whose destructor will automatically release the resource when control leaves the scope of the resource. By putting resources inside objects, we can rely on C++'s automatic destructor invocation to make sure that the resources are released.

- **Resources are acquired and immediately turned over to resource-managing objects.** Smart pointers manages resources. This idea of using objects to manage resources is often called _Resource Acquisition Is Initialization_ (RAII), because it's so common to acquire a resource and initialize a resource-managing object in the same statement.
- **Resource-managing objects use their destructors to ensure that resources are released.** Because destructors are called automatically when an object is destroyed, resources are correctly released, regardless of how control leaves a block.

**Things to Remember**

- To prevent resource leaks, use RAII objects that acquire resources in their constructors and release them in their destructors.
- Two commonly useful RAII classes are `tr1::shared_ptr` and `auto_ptr`. `tr1::shared_ptr` is usually the better choice, because its behavior when copied is intuitive. Copying an `auto_ptr` sets it to `null`.

## Item 14: Think carefully about copying behavior in resource-managing classes.

Every RAII class author must confront the question: what should happen when an RAII object is copied? Most of the time, you'll want to choose one of the following possibilities:

- Prohibit copying.
- Reference-count the underlying resource.
- Copy the underlying resource.
- Transfer ownership to the underlying resource.

**Things to Remember**

- Copying an RAII object entails copying the resources it manages, so the copying behavior of the resource determines the copying behavior of the RAII object.
- Common RAII class copying behaviors are disallowing copying and performing reference counting, but other behaviors are possible.

# Item 15: Provide access to raw resources in resource-managing classes.

**Things to Remember**

Consider this RAII class for fonts that are native to a C API:

```c++
FontHandler getFont();

void releaseFont(FontHandle fh);

class Font {
public:
  explicit Font(FontHandle fh) : f(fh) {}

  ~Font() { releaseFont(f); }

private:
  FontHandle f;
};
```

We can provide the user of `Font` access to the resources either explicitly or implicitly:

_explicit_

```c++
class Font {
public:
  ...
  FontHandle get() const { return f; }
  ...
};
```

_implicit_

```c++
class Font {
public:
  ...
  operator FontHandle() const { return f; }
  ...
};
```

Explicit is safer, implicit can be more natural to the user.

**Things to Remember**

- APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages.
- Access may be via explicit conversion or implicit conversion. In general, explicit conversion is safer, but implicit conversion is more convenient for clients.

# Item 16: Use the same form in corresponding uses of `new` and `delete`.

Whats's wrong with this picture?

```c++
std::string *stringArray = new std::string[100];
...
delete stringArray;
```

The program's behavior is undefined. At the very least, 99 of the 100 `string` objects pointed to by `stringArray` are unlikely to be properly destroyed, because their destructors will probably never be called.

When you use `delete` on a pointer, the only way for `delete` to know whether the array size information is there is for you to tell it. If you use brackets in your use of `delete`, `delete` assumes an array is pointed to. Otherwise, it assumes that a single object is pointed to:

```c++
std::string *stringPtr1 = new std::string;
std::string *stringPtr2 = new std::string[100];
...
delete stringPtr1;
delete [] stringPtr2;
```

**Things to Remember**

- If you use [] in a `new` expression, you must use [] in the corresponding `delete` expression. If you don't use [] in a `new` expression, you mustn't use [] in the corresponding `delete` expression.

# Item 17: store `new`ed objects in smart pointers in standalone statements.

**Things to Remember**

Consider the following call which will compile:

```c++
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```

Surprisingly, although we're using object-managing resources everywhere here, this call may leak resources. It's illuminating to see how.

Before compilers can generate a call to `processWidget`, they have to evaluate the arguments being passed as its parameters. The second argument is just a call to the function `priority`, but the first argument consists of two parts:

- Execution of the expression "`new Widget`".
- A call to the `tr1::shared_ptr` constructor.

Before `processWidget` can be called, then, compilers, must generate code to do these three things:

- Call `priority`.
- Execute `new Widget`.
- Call the `tr1::shared_ptr` constructor.

C++ compilers are granted considerable latitude in determining the order in which these things are to be done. The "`new Widget`" expression must be executed before the `tr1::shared_ptr` constructor can be called, because the result of the expression is passed as an argument to the `tr1::shared_ptr` constructor, but the call to `priority` can be performed first, second, or third. If compilers choose to perform it second, we end up with this sequence of operations:

1. Execute "`new Widget`".
2. Call `priority`.
3. Call the `tr1::shared_ptr` constructor.

But consider what will happen if the call to `priority` yields an exception. In that case, the pointer returned from "`new Widget`" will be lost, because it won't have been stored in the `tr1::shared_ptr` we were expecting would guard against resource leaks. A leak in the call to `processWidget` can arise because an exception can intervene between the time a resource is created and the time that resource is turned over to a resource-managing object.

The way to avoid problems like this is simple: use a separate statement to create the `Widget` and store it in a smart pointer, then pass the smart pointer to `processWidget`:

```c++
std::tr1::shared_ptr<Widget> pw(new Widget);

processWidget(pw, priority());
```

**Things to Remember**

- Store `new`ed objects in smart pointers in standalone statements. Failure to do this can lead to subtle resource leaks when exceptions are thrown.
