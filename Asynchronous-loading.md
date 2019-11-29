`osu-framework` puts a lot of focus on non-blocking loading, striving to never block the interface threads at any point. It is therefore recommended (and in some cases forced) that you use async patterns when loading components.

Preloading components
========

It is recommended that any larger components are preloaded ahead of time. For instance, after a user has arrived on a menu screen, loading could begin on the next screen(s) that will be displayed.

The simplest method to load a component in the background is by calling `LoadComponentAsync` from inside a `Drawable`. This method also exposes a callback which can be used to perform an action when loading completes, which can be useful for showing a newly loading component as soon as it is ready.

Instantly using the component:

```csharp
LoadComponentAsync(new MyComponent(), Add);
```

Preloading for future use:

```csharp
var myComponent = new MyComponent();

LoadComponentAsync(myComponent);

...

Add(myComponent); // note that this will cause a blocking load if the async load has not yet completed.
```

This method returns a `Task` which can be blocked on with `Task.Wait()` if you need custom blocking behaviour.

Asynchronous chaining with [BackgroundDependencyLoader]
========

When a component is loaded, it will attempt to load all nested children that have `ShouldBeAlive == true`. To make use of this chaining, ensure that all loading code is either in the `ctor` or a private method marked with the `[BackgroundDependencyLoader]` attribute. The latter is recommended as it avoids a potential overhead when constructing new instances of the component that aren't nested in an asynchronous load.

```csharp
public class MainComponent : CompositeDrawable
{
    [BackgroundDependencyLoader]
    private void load()
    {
        Child = new NestedComponent();
        // this could also be in the ctor if required, but generally use BDL wherever possible for maximum efficiency.
    }
}

public class NestedComponent : Drawable
{
    [BackgroundDependencyLoader]
    private void load()
    {
        // long running load
    }

}
```

The following call could then be used to load `MainComponent` and `NestedComponent` asynchronously, only adding MainCompoent when the nested tree is completely loaded.

```csharp
LoadComponentAsync(new MainComponent(), Add);
```

Overriding pre-packaged asynchronous behaviour
========

Some classes come with pre-built asynchronous behaviours. One example is `Screen`, which automatically loads a child screen that is `Push`ed if it is not already in a loaded state. Overriding such asynchronous behaviour can be achieved as mentioned above, by manually blocking on a preload call:

```csharp
var blocking = new BlockingScreen();

LoadComponentAsync(blocking).Wait();
screenStack.Push(blocking);
```

Long-running load components (online retrieval etc.)
========

The special attribute `[LongRunningLoad]` exists to mark components which require loads that take a long time (generally anything >100ms). This is usually a drawable which is retrieving a texture (or other content) from a network source. By attaching this to a class, you can ensure that all usage of that class is correctly loaded in an asynchronous context.

There are two important things to note about this attribute:

### Marked components will run in their own segregated thread pool

This avoids any potential of thread pool saturation by network requests or otherwise. Normal components loaded via `LoadComponentAsync` will be unaffected by `LongRunningLoad` components.

### Marked components must be a top-level async load

In order to avoid `LongRunningLoad` components accidentally ending up on the standard async load thread pool, they must always be run *directly* via `LoadComponentAsync`. An exception will be thrown if this is not the case.