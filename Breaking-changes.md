Occasionally we will make changes which require consumers of the framework to make amendments to maintain compatibility. Wherever possible, we maintain compatibility via `[Obsolete]` attributes, but it is encouraged that you leave obsolete warnings turned on an deal with them sooner rather than later. Generally we aim to leave obsolete methods around for 3-6 months after obsoletion.

This page serves to give a list of all breaking/major changes.

# 2019.604.0

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
