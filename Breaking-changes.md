Occasionally we will make changes which require consumers of the framework to make amendments to maintain compatibility. Wherever possible, we maintain compatibility via `[Obsolete]` attributes, but it is encouraged that you leave obsolete warnings turned on, and deal with them sooner rather than later. Generally we aim to leave obsolete methods around for 3-6 months after obsoletion.

This page serves to give a list of all breaking/major changes.

# vNext

## `CachedModelDependencyContainer` models must now only contain read-only fields

Mutating bindables of models attached to a `CachedModelDependencyContainer` can lead to crashes or otherwise unexpected behaviour, and is now disallowed.

# [2020.302.0](https://github.com/ppy/osu-framework/releases/tag/2020.302.0)

## `FrameworkDebugConfig` no longer exposes `PerformanceLogging`

Performance logging is now toggled by opening "Global Statistics" overlay via Ctrl+F2. To manually toggle it, you may access `GameHost.PerformanceLogging`, but note that a change to this bindable will be overridden by toggling the statistics overlay.

## Layout validation and invalidation has been reworked

To better cover edge-case scenarios where layout wasn't properly refreshed, the process of layout invalidation has changed.

### In cases where `Invalidate()` affected a `Cached` member, the following adjustment is necessary:

```diff
public class MyDrawable : Drawable
{
-    private readonly Cached myLayoutValue = new Cached();
-
-    protected override bool OnInvalidate(Invalidation invalidation, Drawable source, bool shallPropagate)
-    {
-        var result = base.OnInvalidate(invalidation, source, shallPropagate);
-
-        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
-            result &= !myLayoutValue.Invalidate();
-
-        return result;
-    }

+    /* can also be a LayoutValue<T> */
+    private readonly LayoutValue myLayoutValue = new LayoutValue(Invalidation.DrawSize | Invalidation.Presence);
+
+    public MyDrawable()
+    {
+        AddLayout(myLayoutValue);
+    }
}
```

### In cases where `InvalidateFromChild()` affected a `Cached` member, the following adjustment is necessary:

```diff
public class MyDrawable : Drawable
{
-    private readonly Cached myLayoutValue = new Cached();
-
-    protected override void InvalidateFromChild(Invalidation invalidation, Drawable source)
-    {
-        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
-            myLayoutValue.Invalidate();
-    }

+    /* can also be a LayoutValue<T> */
+    private readonly LayoutValue myLayoutValue = new LayoutValue(Invalidation.DrawSize | Invalidation.Presence, InvalidationSource.Child);
+
+    public MyDrawable()
+    {
+        AddLayout(myLayoutValue);
+    }
}
```

### In cases where _both_ `Invalidate()` and `InvalidateFromChild()` affected the same `Cached` member, the following adjustment is required:

```diff
public class MyDrawable : Drawable
{
-    private readonly Cached myLayoutValue = new Cached();
-
-    protected override bool OnInvalidate(Invalidation invalidation, Drawable source, bool shallPropagate)
-    {
-        var result = base.OnInvalidate(invalidation, source, shallPropagate);
-
-        if ((invalidation & Invalidation.DrawSize) > 0) /* any invalidation type of your choice */
-            result &= !myLayoutValue.Invalidate();
-
-        return result;
-    }
-
-    protected override void InvalidateFromChild(Invalidation invalidation, Drawable source)
-    {
-        if ((invalidation & Invalidation.Presence) > 0) /* any invalidation type of your choice */
-            myLayoutValue.Invalidate();
-    }

+    /* can also be a LayoutValue<T> */
+    private readonly LayoutValue myLocalLayoutValue = new LayoutValue(Invalidation.DrawSize);
+    private readonly LayoutValue myChildLayoutValue = new LayoutValue(Invalidation.Presence, InvalidationSource.Child);
+
+    public MyDrawable()
+    {
+        AddLayout(myLocalLayoutValue);
+        AddLayout(myChildLayoutValue);
+    }
+
+    protected override void Update()
+    {
+        base.Update();
+
+        /* wherever your re-validation was previously done */
+        if (!myChildLayoutValue.IsValid)
+        {
+            myLocalLayoutValue.Invalidate();
+            myChildLayoutValue.Validate();
+        }
+
+        if (!myLocalLayoutValue.IsValid)
+        {
+            /* your custom validation */
+            myLocalLayoutValue.Validate();
+        }
+    }
}
```

### In cases where custom logic that can't be described as layout was done in `Invalidate()`, the following adjustment is necessary:

```diff
public class MyDrawable : Drawable
{
-    protected override bool OnInvalidate(Invalidation invalidation, Drawable source, bool shallPropagate)
-    {
-        var result = base.OnInvalidate(invalidation, source, shallPropagate);
-
-        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
-        {
-            performCustomAction();
-            result = true;
-        }
-
-        return result;
-    }

+    protected override bool OnInvalidate(Invalidation invalidation, InvalidationSource source)
+    {
+        var result = base.OnInvalidate(invalidation, source);
+
+        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
+        {
+            performCustomAction();
+            result = true;
+        }
+
+        return result;
+    }

    private void performCustomAction()
    {
        /* custom invalidation logic here */
    }
}
```

# [2020.218.0](https://github.com/ppy/osu-framework/releases/tag/2020.218.0)

## `BindableList<T>.ItemsAdded` is now obsolete

The `ItemsAdded` event failed to provide enough context when items are moved around or inserted into the list.

It has been replaced by the `CollectionChanged` [NotifyCollectionChangedEventHandler](https://docs.microsoft.com/en-au/dotnet/api/system.collections.specialized.notifycollectionchangedeventhandler?view=netframework-4.8), which provides context such as the newly-added items and the indices at which the addition took place.

This event is triggered by the following methods with `Action = NotifyCollectionChangedAction.Add`:
```csharp
BindableList<T>.Add(T item)
BindableList<T>.Insert(int index, T item)
BindableList<T>.AddRange(IEnumerable<T> items)
```

The following is an example migration utilising the new `CollectionChanged` event:
```diff
BindableList<int> list = new BindableList<int>();

- list.ItemsAdded += items =>
- {
-     foreach (var item in items)
-         Console.WriteLine($"Added: {item}");
- }

+ list.CollectionChanged += (_, args) =>
+ {
+     switch (args.Action)
+     {
+         case NotifyCollectionChangedAction.Add:
+             foreach (var item in args.NewItems.Cast<int>())
+                 Console.WriteLine($"Added: {item});
+             break;
+     }
+ }
```

Note that `CollectionChanged` should _not_ be used in conjunction with the `ItemsAdded` event.

## `BindableList<T>.ItemsRemoved` is now obsolete

As above, the `ItemsRemoved` revent has also been replaced by the `CollectionChanged` [NotifyCollectionChangedEventHandler](https://docs.microsoft.com/en-au/dotnet/api/system.collections.specialized.notifycollectionchangedeventhandler?view=netframework-4.8).

This event is triggered by the following methods with `Action = NotifyCollectionChangedAction.Remove`:
```csharp
BindableList<T>.Clear()
BindableList<T>.Remove(T item)
BindableList<T>.RemoveRange(int index, int count)
BindableList<T>.RemoveAt(int index)
BindableList<T>.RemoveAll(Predicate<T> match)
```

The following is an example migration utilising the new `CollectionChanged` event:
```diff
BindableList<int> list = new BindableList<int>();

- list.ItemsRemoved += items =>
- {
-     foreach (var item in items)
-         Console.WriteLine($"Removed: {item}");
- }

+ list.CollectionChanged += (_, args) =>
+ {
+     switch (args.Action)
+     {
+         // Handles both clear and remove events
+         case NotifyCollectionChangedAction.Remove:
+             foreach (var item in args.OldItems.Cast<int>())
+                 Console.WriteLine($"Removed: {item});
+             break;
+     }
+ }
```

# [2020.122.0](https://github.com/ppy/osu-framework/releases/tag/2020.122.0)

## InputManager.CreateButtonManagerFor was renamed to InputManager.CreateButtonEventManagerFor

Changed to match other methods and the class name it was creating.

## Various input "end" events now return `void` and can no longer block propagation

Events affected:
```csharp
Drawable.OnDoubleClick()
Drawable.OnDrag()
Drawable.OnDragEnd()
Drawable.OnMouseUp()
Drawable.OnKeyUp()
Drawable.OnJoystickRelease()
IKeyBindingHandler<T>.OnReleased()
```

A drawable may return `false` for an input "begin" event to allow the event to propagate further through the hierarchy. Previously, such a drawable could then return `true` for the input "end" event and prevent the event from propagating to the drawables which the input "begin" event was propagated to.  
This could lead to incorrect implementations where drawables are left in weird states after handling input events.

By returning `void`, the input "end" events can no longer be blocked by other drawables in the hierarchy and are guaranteed to be invoked if their respective input "begin" event was previously invoked.

The following table illustrates the events for which the relationship is satisfied:

|     begin event     |        end event(s)       |
| ------------------- | ------------------------- |
| `OnDragStart()`     | `OnDrag()`, `OnDragEnd()` |
| `OnMouseDown()`     | `OnMouseUp()`             |
| `OnKeyDown()`       | `OnKeyUp()`               |
| `OnJoystickPress()` | `OnJoystickRelease()`     |
| `OnPressed()`       | `OnReleased()`            |

# [2020.109.0](https://github.com/ppy/osu-framework/releases/tag/2020.109.0)

## The namespace `osu.Framework.MathUtils` has been removed

All existing utility classes have been moved to the namespace `osu.Framework.Utils`.

# [2019.1224.0](https://github.com/ppy/osu-framework/releases/tag/2019.1224.0)

## `TextBox` and all related components are now abstract

`BasicTextBox` is provided as a drop-in replacement that comes with the framework design language.

## `PasswordTextBox` has been renamed 

Renamed to `BasicPasswordTextBox` and comes with the framework design language.

# [2019.1211.1](https://github.com/ppy/osu-framework/releases/tag/2019.1211.1)

## The value provided in the constructor for `Bindable<T>` is now used as both the initial and default value

The following lines of code are now identical.
```csharp
var bindable1 = new Bindable<int>(10) { Default = 10 };
var bindable2 = new Bindable<int>(10);
```

# [2019.1210.1](https://github.com/ppy/osu-framework/releases/tag/2019.1210.1)

## PerformanceLogging was moved to DebugConfig

As seen in https://github.com/ppy/osu/issues/6795, some users are turning on performance logging and leaving it on. This is not intended, as it adds a noticeable overhead to retrieve and write the stack traces to disk.

This change allows the setting to reset each game execution, rather than be saved to a user's configuration file.

# [2019.1121.0](https://github.com/ppy/osu-framework/releases/tag/2019.1121.0)

## .NET Standard 2.1

osu!framework now targets .NET Standard 2.1. Consumers of the framework will need to install the [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) and change their projects to target `netstandard2.1`/`netcoreapp3.0`.

## `WebRequest.ResponseString` and `WebRequest.ResponseData` properties were converted to methods

`GetResponseString` and `GetResponseData` methods are provided as a replacement. This change was done in order to indicate that these operations are expensive(they do memory allocations in the heap on each call).

# [2019.1104.0](https://github.com/ppy/osu-framework/releases/tag/2019.1104.0)

## `Button` has been made abstract

`BasicButton` is provided as a drop-in replacement that comes with the framework design language.

# [2019.921.0](https://github.com/ppy/osu-framework/releases/tag/2019.921.0)

## `Quad.ConservativeArea` has been removed

It was inaccurate for certain `Quad`s. Use `Area` as a replacement.

# [2019.821.0](https://github.com/ppy/osu-framework/releases/tag/2019.821.0)

## `SpriteText.UseFixedWidthForCharacter` has been removed

Provide an array of `char`s as `SpriteText.FixedWidthExcludeCharacters` instead.

## `SpriteText.GetTextureForCharacter` and `SpriteText.GetFallbackTextureForCharacter` have been removed.

Glyphs must now always be retrieved through an `ITexturedGlyphLookupStore`. The `SpriteText.CreateTextBuilder()` method is provided to allow overriding the store which glyphs are retrieved from.

## `BlendingModes` are now obsoleted

Please switch to using `BlendingParameters` static properties instead, whcih provide the same functionality. This change was made to expose full control over blend equations and simplify the blending class structure, which previously spanned three related types.

# [2019.809.0](https://github.com/ppy/osu-framework/releases/tag/2019.809.0)

## `Cached` is now a class and requires object initialisation

Usages of `Cached` without initialising (via `new Cached()`) will need to be updated.

## `Animation` no longer supports `AutoSizeAxes`

`Animation` will automatically auto-size on axes which are not relative sized (via `RelativeSizeAxes`) and for which `Size` has not been set.

# [2019.726.0](https://github.com/ppy/osu-framework/releases/tag/2019.726.0)

## `Path` now supports `AutoSizeAxes` and is set to auto-size in both axes by default

There are now three modes of operation for paths:
```
// Auto size
new Path()

// Static size
new Path
{
    AutoSizeAxes = Axes.None,
    Size = new Vector2(100)
}

// Relative size
new Container
{
    Size = new Vector2(100),
    new Path
    {
        AutoSizeAxes = Axes.None,
        RelativeSizeAxes = Axes.Both
    }
}
```

# [2019.702.0](https://github.com/ppy/osu-framework/releases/tag/2019.702.0)

## `InputKey` enum names for extra mouse buttons have been renamed.

```
MouseButtonX -> ExtraMouseButtonX
```

# [2019.628.0](https://github.com/ppy/osu-framework/releases/tag/2019.628.0)

## `Menu` and several related components have been made abstract

For each component, there are two ways to resolve resulting errors:

1. `ContextMenuContainer`
    1. `BasicContextMenuContainer` is provided as a drop-in replacement that comes with the framework design language.
    2. Implement your own local `ContextMenuContainer`. The following is consumable code that matches the removed implementation. Rename it to suit your project and update your references:  
        ```csharp
        public class MyContextMenuContainer : ContextMenuContainer
        {
            // If you have a custom menu, provide it here.
            protected override Menu CreateMenu() => new BasicMenu(Direction.Vertical);
        }
        ```
2. `Menu`
    1. `BasicMenu` is provided as a drop-in replacement that comes with the framework design language.
    2. Implement your own local `Menu`. The following is consumable code that matches the removed implementation. Rename it to suit your project and update your references:  
        ```csharp
        public class MyMenu : Menu
        {
            public MyMenu(Direction direction, bool topLevelMenu = false)
                : base(direction, topLevelMenu)
            {
            }
        
            protected override Menu CreateSubMenu() => new MyMenu(Direction.Vertical);
        
            protected override DrawableMenuItem CreateDrawableMenuItem(MenuItem item) => new MyDrawableMenuItem(item);
        
            // If you have a custom scroll container, provide it here.
            protected override ScrollContainer<Drawable> CreateScrollContainer(Direction direction) => new BasicScrollContainer(direction);
        
            private class MyDrawableMenuItem : DrawableMenuItem
            {
                public MyDrawableMenuItem(MenuItem item)
                    : base(item)
                {
                }
        
                protected override Drawable CreateContent() => new SpriteText
                {
                    Anchor = Anchor.CentreLeft,
                    Origin = Anchor.CentreLeft,
                    Padding = new MarginPadding(5),
                    Font = new FontUsage(size: 17),
                };
            }
        }
        ```
3. `DropdownMenu` / `DropdownMenuItem`. This is only applicable if you already implement a custom `Dropdown<T>`.
    1. The following is consumable code that matches the removed implementation. Rename it to suit your project and update your references:
        ```csharp
        public class MyDropdown<T> : Dropdown<T>
        {
            // If you have a custom header, provide it here.
            protected override DropdownHeader CreateHeader() => null;
        
            protected override DropdownMenu CreateMenu() => new MyDropdownMenu();
        
            private class MyDropdownMenu : DropdownMenu
            {
                // If you have a custom menu, provide it here.
                protected override Menu CreateSubMenu() => new BasicMenu(Direction);
        
                // If you have a custom scroll container, provide it here.
                protected override ScrollContainer<Drawable> CreateScrollContainer(Direction direction) => new BasicScrollContainer(direction);
        
                protected override DrawableDropdownMenuItem CreateDrawableDropdownMenuItem(MenuItem item) => new MyDrawableDropdownMenuItem(item);
        
                private class MyDrawableDropdownMenuItem : DrawableDropdownMenuItem
                {
                    public MyDrawableDropdownMenuItem(MenuItem item)
                        : base(item)
                    {
                    }
        
                    // If you have custom content, provide it here.
                    protected override Drawable CreateContent() => new SpriteText
                    {
                        Anchor = Anchor.CentreLeft,
                        Origin = Anchor.CentreLeft,
                        Padding = new MarginPadding(5),
                        Font = new FontUsage(size: 17),
                    };
                }
            }
        }
        ```

## `DebugSetting.ActiveGCMode` no longer exists

If you were manually setting this, you can do so via [.NET methods](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/latency?view=netframework-4.8).

# [2019.614.0](https://github.com/ppy/osu-framework/releases/tag/2019.614.0)

## `ScrollContainer<T>` is now abstract (and `ScrollContainer` no longer exists)

In a push to make the provided framework components design agnostic, `ScrollContainer` is no longer provided. As a framework consumer you have a few options to resolve this:

- Use `BasicScrollContainer`

The framework now provides `BasicScrollContainer` and `BasicScrollContainer<T>`. These are drop-in replacements but come with the framework design language, so it is only recommended as a temporary solution.

- Implement your own local `ScrollContainer`

Here is consumable code to provide a local implementation that matches the removed one. Rename to suit your project and update your references:

```csharp
public class MyScrollContainer : ScrollContainer<Drawable>
{
    public MyScrollContainer(Direction scrollDirection = Direction.Vertical)
        : base(scrollDirection)
    {
    }

    protected override ScrollbarContainer CreateScrollbar(Direction direction) => new MyScrollbar(direction);

    protected class MyScrollbar : ScrollbarContainer
    {
        private const float dim_size = 10;

        private readonly Color4 hoverColour = Color4.White;
        private readonly Color4 defaultColour = Color4.Gray;
        private readonly Color4 highlightColour = Color4.Black;

        private readonly Box box;

        public MyScrollbar(Direction scrollDir)
            : base(scrollDir)
        {
            Colour = defaultColour;

            Blending = BlendingMode.Additive;

            CornerRadius = 5;

            const float margin = 3;

            Margin = new MarginPadding
            {
                Left = scrollDir == Direction.Vertical ? margin : 0,
                Right = scrollDir == Direction.Vertical ? margin : 0,
                Top = scrollDir == Direction.Horizontal ? margin : 0,
                Bottom = scrollDir == Direction.Horizontal ? margin : 0,
            };

            Masking = true;
            Child = box = new Box { RelativeSizeAxes = Axes.Both };

            ResizeTo(1);
        }

        public override void ResizeTo(float val, int duration = 0, Easing easing = Easing.None)
        {
            Vector2 size = new Vector2(dim_size)
            {
                [(int)ScrollDirection] = val
            };
            this.ResizeTo(size, duration, easing);
        }

        protected override bool OnHover(HoverEvent e)
        {
            this.FadeColour(hoverColour, 100);
            return true;
        }

        protected override void OnHoverLost(HoverLostEvent e)
        {
            this.FadeColour(defaultColour, 100);
        }

        protected override bool OnMouseDown(MouseDownEvent e)
        {
            if (!base.OnMouseDown(e)) return false;

            //note that we are changing the colour of the box here as to not interfere with the hover effect.
            box.FadeColour(highlightColour, 100);
            return true;
        }

        protected override bool OnMouseUp(MouseUpEvent e)
        {
            if (e.Button != MouseButton.Left) return false;

            box.FadeColour(Color4.White, 100);

            return base.OnMouseUp(e);
        }
    }
}
```

# [2019.611.0](https://github.com/ppy/osu-framework/releases/tag/2019.611.0)

## `LinearVertexBuffer` and `QuadVertexBuffer` can no longer be constructed outside of osu!framework.

For existing code, `LinearBatch(size, 1)` and `QuadBatch(size, 1)` may be used to replace vertex buffer usages. Code must be updated to always invoke `batch.Add(vertex)` with the vertices that should be drawn.

`VertexBuffer` may still be derived for custom implementations.

## VisibilityContainer now exposes visibility via a bindable

`VisibilityContainer` no longer implements `IStateful<Visibility>`, instead exposing a single `Bindable<Visibility> State`.

# [2019.606.0](https://github.com/ppy/osu-framework/releases/tag/2019.606.0)

## `Texture.DrawTriangle()` and `Texture.DrawQuad()` have been removed ([#2475](https://github.com/ppy/osu-framework/pull/2475))

Use `DrawNode.DrawTriangle()` and `DrawNode.DrawQuad()` instead.

# [2019.604.0](https://github.com/ppy/osu-framework/releases/tag/2019.604.0)

## Global `TrackStore` and `ResourceStore`s can no longer access resources that are out of their respective namespaces.

Previously, it was possible to access any resource as a part of `TrackStore` and `SampleStore` via their `Get` methods because the entire resource store would be nested inside them. 

This nesting has been removed, so you can now only access files that are part of their specified namespaces (folders). 

* For `TrackStore`, this will be the `Tracks` namespace. 
* For `SampleStore`, this will be the `Samples` namespace.

## Audio stores have been renamed

To bring naming in-line with the purpose of audio stores, the following classes have been renamed:

`SampleManager` -> `SampleStore`  
`TrackManager` -> `TrackStore`

In line with this, return values of `GetSampleStore` and `GetTrackStore` now return `ISampleStore` and `ITrackStore` respectively, hiding the underlying manager logic.

The public variables relating to these stores have been renamed to reflect this change:

`AudioManager.Track` -> `AudioManager.Tracks`  
`AudioManager.Sample` -> `AudioManager.Samples`

## Virtual tracks must be retrieved from `ITrackStore`

`ITrackStore` now has a `GetVirtual` method which will create a virtual track. This will handle adding the track correctly to the audio subsystem. If one wishes to create a custom type of virtual tracks (a super-edge-case) you could implement the `ITrackStore` interface and manually add your component to `AudioManager` via `AddItem`.

# [2019.523.0](https://github.com/ppy/osu-framework/releases/tag/2019.523.0)

## TabControl can now select nothing [#2430](https://github.com/ppy/osu-framework/pull/2430)

While this does not match most OS implementations of a tab control, this was deemed useful for o!f usage scenarios.

Previously, this code would throw an exception.

```csharp
tabControl.Current.Value = null;
```

Now it is allowed.

## Dropdown can now select nothing [#2428](https://github.com/ppy/osu-framework/pull/2428)

Previously, this code would throw an exception.

```csharp
dropdownMenu.Current.Value = null;
```

Now it is allowed. Note that implementations of dropdown should be updated to handle this (common scenario is that the `DropdownHeader` would not correctly handle a `string.Empty` case if it was using AutoSize in the Y axis).

## `TabItem.IsRemovable` is true by default [#2425](https://github.com/ppy/osu-framework/pull/2425)

This felt like a more sane default. Any existing usage of custom `TabItem`s where `IsRemovable` was overridden, or any existing usage where removal is explicitly *not* wanted need to be updated.

# [2019.514.0](https://github.com/ppy/osu-framework/releases/tag/2019.514.0)

## `Game.FrameStatisticsMode` is now a bindable named `Game.FrameStatistics` [#2399](https://github.com/ppy/osu-framework/pull/2399)

This allows consumers to hook value changed events so they can, for instance, save the (framework) FPS display's state to config and handle hotkey-based changes.

## Visual `TestCase`s are now `TestScene`s [#2365](https://github.com/ppy/osu-framework/pull/2365)

The term `TestCase` is used by testing frameworks to denote single methods inside a test class. We were using it on the class itself as a prefix, which got quite confusing. This resolves the conflict and feels more correct as these are visual tests – ie. "scenes".

Note that the `TestCase` prefix is still supported by tooling for the time being. We still recommend you update (via a simple rename) as soon as possible.

# [2019.427.0](https://github.com/ppy/osu-framework/releases/tag/2019.427.0)

## `Drawable.ApplyDrawNode()` has been removed [#2314](https://github.com/ppy/osu-framework/pull/2314)

The direction of application of `DrawNode` states has been inversed.

Previously:
```csharp
class MyCustomDrawable : Drawable
{
    private bool state;

    protected override DrawNode CreateDrawNode() => new CustomDrawNode();

    protected override void ApplyDrawNode(DrawNode node)
    {
        base.ApplyDrawNode(node);

        var n = (CustomDrawNode)n;
        
        n.State = state; 
    }

    private class CustomDrawNode : DrawNode
    {
        public bool State;
    }
}
```

Now:
```csharp
class MyCustomDrawable : Drawable
{
    private bool state;

    protected override DrawNode CreateDrawNode() => new CustomDrawNode(this);

    private class CustomDrawNode : DrawNode
    {
        protected new MyCustomDrawable Source => (MyCustomDrawable)base.Source;

        private bool state;

        public CustomDrawNode(MyCustomDrawable source)
            : base(source)
        {
        }

        public override void ApplyState()
        {
            base.ApplyState();

            state = Source.state;
        }
    }
}
```

## The namespace of `EdgeEffectParameters` and `EdgeEffectType` has changed [#2314](https://github.com/ppy/osu-framework/pull/2314)

They now reside in `osu.Framework.Graphics.Effects`.

# [2019.327.0](https://github.com/ppy/osu-framework/releases/tag/2019.327.0)

## Font fallback order changes [#2296](https://github.com/ppy/osu-framework/pull/2296)

Games adding their own fonts can now do so directly to `Game.Fonts`. Fallback (framework-side) fonts are now always at the lowest priority.

Previously:
```csharp
// this completely overrides the framework default. will need to change once we make a proper FontStore.
dependencies.Cache(Fonts = new FontStore(new GlyphStore(Resources, @"Fonts/FontAwesome")));
Fonts.AddStore(new GlyphStore(Resources, @"Fonts/osuFont"));
```

now:
```csharp
Fonts.AddStore(new GlyphStore(Resources, @"Fonts/osuFont"));
```

## FontAwesome and SpriteIcon is provided by the framework [#2293](https://github.com/ppy/osu-framework/pull/2293)

If you manually added either of these to your project, please remove them.

Usage:
```csharp
new SpriteIcon
{
    Icon = FontAwesome.Refresh,
};
```

# [2019.319.0](https://github.com/ppy/osu-framework/releases/tag/2019.319.0)

## Parameter order of `AddUntilStep` / `AddWaitStep` reversed [#2256](https://github.com/ppy/osu-framework/pull/2256)

In order to standardise parameter order, these two methods' parameters have been reversed. Old parameter order methods are available temporarily (as `[Obsolete]`) to ease migration.

## New attribute `[SetUpSteps]` added [#2266](https://github.com/ppy/osu-framework/pull/2266)

Finally allows steps to be added that will run before every `[Test]` method.

# [2019.221.0](https://github.com/ppy/osu-framework/releases/tag/2019.221.0)

## Implicit operator removed from `Bindable` [#2152](https://github.com/ppy/osu-framework/pull/2152)

In order to avoid accidental misuse / misunderstandings, `.Value` must always be added to `Bindable` when requesting its value. One such case which used to be confusing:

```csharp
Bindable<Drawable> bindableDrawable = new Bindable<Drawable>();

if (bindableDrawable != null)
{
    // true
}

if (bindableDrawable.Value != null)
{
    // false
}

Drawable drawable = null;

if (bindableDrawable == drawable)
{
    // ???
}
```

## Bindable.ValueChanged now provides `ValueChangedEvent` [#2012](https://github.com/ppy/osu-framework/pull/2012)

It is now possible to access the old value. This was a common requirement in bindable usage which we had issues with until now.

```csharp

bindable.ValueChanged += e =>
{
    Console.WriteLine($"Value changed from {e.OldValue} to {e.NewValue}");
}
```

## Introduction of `FontUsage` [#2043](https://github.com/ppy/osu-framework/pull/2043)

While old methods of setting font attributes will still work, they are now marked as `[Obsoleted]`. You should use `Font = new FontUsage(size: 200)` going forward. For convenience, you can adjust existing fonts like so:

```csharp
var font = new FontUsage(family: "SourceCodePro");

var fontWithSize = font.With(size: 24);
```
