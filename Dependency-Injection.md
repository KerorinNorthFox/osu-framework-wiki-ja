In osu-framework, we support dependency injection at a `Drawable` level. Internally, this is done via `DependencyContainer`s, which are passed down the hierarchy and can be overridden at any point for further customisation (or replacement) by a child.

The general usage for this is to fulfill a dependency that can come from a parent (potentially many levels above the point of usage). It is important to understand the general concept of [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) before reading on.

There are a few ways dependencies can be `cached` (stored) and `resolved` (fetched):

## Using `[Cached]` and `[Resolved]` attributes

This is the simplest implementation.

```csharp
/// <summary>
/// A class which caches something for use by children.
/// </summary>
public class MyGame : Game
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
public class MyComponent : CompositeDrawable
{
    [Resolved]
    protected MyStore FetchedStore { get; set; }

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
/// An class which is to be cached via DI.
/// </summary>
public class MyStore
{
    public string GetAwesomeThing() => "awesome thing!";
}
```

Members marked with either of these attributes are cached or resolved in their respective classes before the [`[BackgroundDependencyLoader]`](#using-[BackgroundDependencyLoader]-to-resolve) method is run.

## Using `[BackgroundDependencyLoader]` to resolve

This can be useful if you want to ensure everything happens in the (potentially asynchronous) `load` method.

```csharp
/// <summary>
/// A class which caches something for use by children.
/// </summary>
public class MyGame : Game
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
public class MyComponent : CompositeDrawable
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

## Using `CreateChildDependencies` to cache

Some more advanced scenarios may require use of this method instead of the `[Cached]` attribute, such as if late initialisation of the cacheable objects is required.

```csharp
/// <summary>
/// A class which caches something for use by children.
/// </summary>
public class MyGame : Game
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
