Occasionally we will make changes which require consumers of the framework to make amendments to maintain compatibility. Wherever possible, we maintain compatibility via `[Obsolete]` attributes, but it is encouraged that you leave obsolete warnings turned on an deal with them sooner rather than later. Generally we aim to leave obsolete methods around for 3-6 months after obsoletion.

This page serves to give a list of all breaking/major changes.

# vNext

## .NET Standard 2.1

osu!framework has been changed to target .NET Standard 2.1. Consumers of the framework will need to install the [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) and change their projects to target `netstandard2.1`/`netcoreapp3.0`.

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
