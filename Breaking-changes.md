Occasionally we will make changes which require consumers of the framework to make amendments to maintain compatibility. Wherever possible, we maintain compatibility via `[Obsolete]` attributes, but it is encouraged that you leave obsolete warnings turned on an deal with them sooner rather than later. Generally we aim to leave obsolete methods around for 3-6 months after obsoletion.

This page serves to give a list of all breaking changes.

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