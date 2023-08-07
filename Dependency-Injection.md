In osu!framework, we support dependency injection at a `Drawable` level. Internally, this is done via `DependencyContainer`s, which are passed down the hierarchy and can be overridden at any point for further customisation (or replacement) by a child.

The general usage for this is to fulfill a dependency that can come from a parent (potentially many levels above the point of usage). It is important to understand the general concept of [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) before reading on.

# Implementation

osu!framework's dependency injection mechanism heavily leans on C# attributes, namely `[Cached]`, `[Resolved]`, and `[BackgroundDependencyLoader]`. Setting the dependencies up is done via one of two pathways: source generation and reflection. Understanding this is key, as the source generation pathway benefits from compile-time optimisations, but requires consumers to adjust their code accordingly.

## Source generation

Since the 2022.1126.0 release, the primary supported implementation of dependency injection relies on [source generators](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview). The primary implications of this for framework consumers are as follows:

- For the source generator-based dependency injection to work, `Drawable` classes must be `partial` so that the source generator can inject the DI machinery into the class. Non-compliant drawables will raise the [`OFSG001`](https://github.com/ppy/osu-framework/blob/e5f1aabc22f2f149f48c4fd9ca7c6f8381b00ec0/osu.Framework.SourceGeneration/Analysers/DiagnosticRules.cs#L14-L21) code inspection.
- In more complicated custom DI usages, if it is desired to `.Inject()` dependencies into a custom non-drawable class, it must implement the marker [`IDependencyInjectionCandidate`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Allocation/IDependencyInjectionCandidate.cs) interface.

The implementation of the source generator can be viewed [here](https://github.com/ppy/osu-framework/blob/master/osu.Framework.SourceGeneration/Generators/Dependencies/DependencyInjectionSourceGenerator.cs).

## Reflection

The original, legacy implementation of dependency injection heavily uses reflection. It will be used if user drawables are not marked `partial`, as the source generator cannot attach its own code to such drawables.

Since the source generator pathway was introduced, this implementation is supported for backwards compatibility, but generally not recommended for new projects.

# Storing and retrieving dependencies

There are a few ways dependencies can be **cached** (stored) and **resolved** (retrieved):

## `[Cached]` on drawable members

This is the simplest implementation.

```csharp
/// <summary>
/// A class which caches something for use by children.
/// </summary>
public partial class MyGame : Game
{
    [Cached]
    protected readonly MyStore Store = new MyStore();

    public MyGame()
    {
        Child = new Container
        {
            RelativeSizeAxes = Axes.Both,
            Child = new Container
            {
                RelativeSizeAxes = Axes.Both,
                Child = new Container
                {
                    RelativeSizeAxes = Axes.Both,
                    Child = new MyComponent()
                }
            },
        };
    }
}

/// <summary>
/// A component that consumed the cached class.
/// </summary>
public partial class MyComponent : CompositeDrawable
{
    [Resolved]
    protected MyStore FetchedStore { get; private set; } = null!;

    protected override void LoadComplete()
    {
        base.LoadComplete();

        InternalChild = new SpriteText
        {
            Text = FetchedStore.GetAwesomeThing()
        };
    }
}

/// <summary>
/// A class which is to be cached via DI.
/// </summary>
public class MyStore
{
    public string GetAwesomeThing() => "awesome thing!";
}
```

Members marked with either of these attributes are cached or resolved in their respective classes before the [`[BackgroundDependencyLoader]`](#using-BackgroundDependencyLoader-to-resolve)-annotated method is run.

## Using `[BackgroundDependencyLoader]` to resolve

This can be useful if you want to ensure everything happens in the (potentially asynchronous) `load()` method.

```csharp
/// <summary>
/// A class which caches something for use by children.
/// </summary>
public partial class MyGame : Game
{
    [Cached]
    protected readonly MyStore Store = new MyStore();

    public MyGame()
    {
        Child = new Container
        {
            RelativeSizeAxes = Axes.Both,
            Child = new Container
            {
                RelativeSizeAxes = Axes.Both,
                Child = new Container
                {
                    RelativeSizeAxes = Axes.Both,
                    Child = new MyComponent()
                }
            },
        };
    }
}

/// <summary>
/// A component that consumed the cached class.
/// </summary>
public partial class MyComponent : CompositeDrawable
{
    [BackgroundDependencyLoader]
    private void load(MyStore store)
    {
        InternalChild = new SpriteText
        {
            Text = store.GetAwesomeThing()
        };
    }
}

/// <summary>
/// An class which is to be cached via DI.
/// </summary>
public class MyStore
{
    public string GetAwesomeThing() => "awesome thing!";
}
```

## Using `CreateChildDependencies()` to cache

Some more advanced scenarios may require use of this method instead of the `[Cached]` attribute, such as if late initialisation of the cacheable objects is required.

```csharp
/// <summary>
/// A class which caches something for use by children.
/// </summary>
public partial class MyGame : Game
{
    protected MyStore Store;

    public MyGame()
    {
        Child = new Container
        {
            RelativeSizeAxes = Axes.Both,
            Child = new Container
            {
                RelativeSizeAxes = Axes.Both,
                Child = new Container
                {
                    RelativeSizeAxes = Axes.Both,
                    Child = new MyComponent()
                }
            },
        };
    }

    protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent)
    {
        var dependencies = new DependencyContainer(base.CreateChildDependencies(parent));
        dependencies.Cache(Store = new MyStore());
        return dependencies;
    }
}
```

Note that the `DependencyContainer` class exposes two methods for caching dependencies:

- `.Cache()` will always cache the dependency using its *runtime, most derived type*. The implications of this are demonstrated by the following example:

    ```csharp
    public abstract class BaseDependency { }
    public class DerivedDependency : BaseDependency { }

    public partial class DependencyProvider
    {
        private BaseDependency dependency;

        protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent)
        {
            var dependencies = new DependencyContainer(base.CreateChildDependencies(parent));
            dependencies.Cache(dependency = new DerivedDependency());
            return dependencies;
        }
    }

    public partial class DependencyConsumer
    {
        [Resolved]
        private BaseDependency baseDependency { get; set; }       // WRONG - will fail at runtime

        [Resolved]
        private DerivedDependency derivedDependency { get; set; } // OK
    }
    ```

- `.CacheAs<T>()` will cache the dependency using its *declared* type, as demonstrated by the following example:

    ```csharp
    public abstract class BaseDependency { }
    public class DerivedDependency : BaseDependency { }

    public partial class DependencyProvider
    {
        private BaseDependency dependency;

        protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent)
        {
            var dependencies = new DependencyContainer(base.CreateChildDependencies(parent));
            dependencies.CacheAs(dependency = new DerivedDependency());
            return dependencies;
        }
    }

    public partial class DependencyConsumer
    {
        [Resolved]
        private BaseDependency baseDependency { get; set; }       // OK

        [Resolved]
        private DerivedDependency derivedDependency { get; set; } // WRONG - will fail at runtime
    }
    ```

## `[Cached]` on drawable classes

Drawable classes themselves can be annotated with the `[Cached]` attribute. In that case, the attribute is interpreted such that all instances of the drawable class will cache themselves to all of their children.

The caching will use the type _at the point of declaration_. To illustrate, given the following structure:

```csharp
[Cached]
public partial class A : Drawable { }
                      
public partial class B : A { }
```

the following things will happen:

- Instances of `A` will cache themselves to their children using type `A`.
- Instances of `B` will cache themselves to their children using type `A`.
- Instances of `B` will **not** cache themselves to their children using type `B`. For that to happen, the `[Cached]` attribute would have to be repeated on type `B`.

## `[Cached]` on interfaces implemented by a drawable class

A variant of type-based caching above is available via interfaces. Interfaces can be annotated with `[Cached]`; every `Drawable` will cache itself to its children using every interface type annotated with `[Cached]` that it implements. As an example:

```csharp
[Cached]
public interface IFirstInterface { }

[Cached]
public interface ISecondInterface { }

public interface IThirdInterface { }

public partial class Dependency : Drawable, IFirstInterface, IThirdInterface { }
```

all instances of `Dependency`:

- will cache themselves to children as `IFirstInterface`,
- will cache themselves to children as `ISecondInterface` (transitively via `IThirdInterface`),
- will **not** cache themselves to children as `IThirdInterface` (as, analogously to classes, `[Cached]` is only valid on types it is explicitly put on)