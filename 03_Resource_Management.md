## Item 13: Use objects to manage resources.

To make sure that an acquired resource is always released, we need to put that resource inside an object whose destructor will automatically release the resource when control leaves the scope of the resource. By putting resources inside objects, we can rely on C++'s automatic destructor invocation to make sure that the resources are released.

- **Resources are acquired and immediately turned over to resource-managing objects.** Smart pointers manages resources. This idea of using objects to manage resources is often called _Resource Acquisition Is Initialization_ (RAII), because it's so common to acquire a resource and initialize a resource-managing object in the same statement.
- **Resource-managing objects use their destructors to ensure that resources are released.** Because destructors are called automatically when an object is destroyed, resources are correctly released, regardless of how control leaves a block.

**Things to Remember**

- To prevent resource leaks, use RAII objects that acquire resources in their constructors and release them in their destructors.
- Two commonly useful RAII classes are `tr1::shared_ptr` and `auto_ptr`. `tr1::shared_ptr` is usually the better choice, because its behavior when copied is intuitive. Copying an `auto_ptr` sets it to `null`.
