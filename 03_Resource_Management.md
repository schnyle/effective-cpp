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

- APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages.
- Access may be via explicit conversion or implicit conversion. In general, explicit conversion is safer, but implicit conversion is more convenient for clients.
