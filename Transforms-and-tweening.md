A relatively extensive toolchain exists for applying transforms to `Drawable`s. This includes not only the ability to perform common visual adjustments (fade, scale, rotate, colour), but also the ability to arbitrarily perform transforms on any field/property belonging to the `Drawable`.

Internally, transforms are a state machine which is build using `TransformSequence`. Multiple transforms can be chained and nested. This page attempts to provide a starting point for understanding how transforms can be used, but there should be plenty of room for exploration beyond this guide via experimentation.

## Basic operations

A simple example which will fade a box into existence:

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(500);
```

## Chaining

Transforms can be chained in various ways. To run multiple transforms of different types at the same time value, simply chain the calls:

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(500)
   .ScaleTo(new Vector2(2))
   .RotateTo(90); // all three of these will be run in parallel.
```

To play one transform after another finishes, use `.Then()`:

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(500).Then().FadeOut(500); // fade in then out, over 1 second.
```

## Offsets and delays

To add a delay to a sequence, you can use `.Delay(ms)`:

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(500).Then().Delay(200).FadeOut(500); // fade in then out, over 1.2 seconds, with a pause at full opacity.
```

For more complex cases, `BeginDelayedSequence` and `BeginAbsoluteSequence` can help to organise things:

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

using (box.BeginAbsoluteSequence(2000)) // nested calls will run from absolute clock time of 2000ms. by default this applies to all children too.
{
    box.FadeIn(1000); 

    using (box.BeginDelayedSequence(1000)) // nested calls will be delayed by 1000ms. by default this applies to all children too.
        box.FadeOut(1000); 
}
```

## Looping

// TODO

## Applying to arbitrary members (fields/properties)

All specific transform methods such as `.FadeIn(...)`/`.FadeColour(...)` are [methods defined in extension classes](https://github.com/ppy/osu-framework/blob/4dbdb0039c1a1e802dfdaab94698f6d9485e6325/osu.Framework/Graphics/TransformableExtensions.cs#L487-L741) that delegate to the core method `.TransformTo()`, which accepts the name of the member to transform/tween, and the remaining arguments for transforming (duration, easing, etc.).

And that doesn't apply only to the `Drawable` base class, you can apply transforms to any member belonging to your class as long as it inherits `Drawable`, and as long as the member is not `static`, and not a `readonly` field or getter-only/setter-only property:

```csharp
public class SpecialBox : Box
{
    public double SpecialProperty { get; set }
}

var box = new SpecialBox { Size = new Vector2(50), SpecialProperty = 1.0 }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.TransformTo(nameof(SpecialProperty), 10.0, 1000); // transform SpecialProperty from current value to value 10.0, in 1 second.
```

For wrapping it in an extensions class:

```csharp
public static class SpecialBoxExtensions
{
    public static TransformSequence<T> TweenSpecialPropertyTo<T>(this T specialBox, double newValue, double duration, Easing easing = Easing.None)
        where T : SpecialBox
        => specialBox.TweenSpecialPropertyTo(newValue, duration, new DefaultEasingFunction(easing));

    public static TransformSequence<T> TweenSpecialPropertyTo<T>(this TransformSequence<T> t, double newValue, double duration, Easing easing = Easing.None)
        where T : SpecialBox
        => specialBox.TweenSpecialPropertyTo(newValue, duration, new DefaultEasingFunction(easing));

    public static TransformSequence<T> TweenSpecialPropertyTo<T, TEasing>(this T specialBox, double newValue, double duration, in TEasing easing)
        where T : SpecialBox
        where TEasing : IEasingFunction
        => specialBox.TransformTo(nameof(SpecialBox.SpecialProperty), newValue, duration, easing);

    public static TransformSequence<T> TweenSpecialPropertyTo<T, TEasing>(this TransformSequence<T> t, double newValue, double duration, in TEasing easing)
        where T : SpecialBox
        where TEasing : IEasingFunction
        => t.Append(o => o.TweenSpecialPropertyTo(newValue, duration, easing));
}
```

```csharp
box.TweenSpecialPropertyTo(10.0, 1000);
```
```csharp
box.FadeInFromZero(500)
   .TweenSpecialPropertyTo(10.0, 1000)
   .Then().FadeOut(500);
```

## Manual interpolation

In cases where transforms are to be run every frame, it is highly recommended to use interpolation instead:

```csharp
public override void Update()
{
    // useful if targetWidth is a moving target which need to be tracked, for example.
    box.Width = Interpolation.ValueAt(Clock.ElapsedFrameTime, box.Width, targetWidth, 0, 200, Easing.OutQuint);
}
```

## Interrupting transforms

Transforms can be interrupted during their application, by finishing them immediately on current time, or removing them and leaving the transformed property at its current state, or rewinding it back to the beginning of the transform.

### Finishing transforms immediately

To finish transforms immediately, use `FinishTransforms` on the drawable whose transforms are still being processed.

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(1000);

// after 0.5 seconds, finish transforms applied to the box.
Scheduler.AddDelay(() => 
{
    // by default, this only applies to transforms applied to the box itself,
    // pass `true` to finish transforms of children as well.
    box.FinishTransforms();
}, 500);
```

### Removing transforms

To remove transforms, use `ClearTransforms` on the drawable whose transforms are still being processed.

```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(1000);

// after 0.5 seconds, remove transforms from the box.
Scheduler.AddDelay(() => 
{
    // by default, this only applies to transforms applied to the box itself,
    // pass `true` to clear transforms of children as well.
    box.ClearTransforms();
}, 500);
```

To remove transforms starting at a certain time, use `ClearTransformsAfter` instead.

```csharp
box.FadeInFromZero(500).Then().FadeOut(500);

// after 0.5 seconds, remove transforms at current time from the box.
Scheduler.AddDelay(() =>
{
    // by default, this only applies to transforms applied to the box itself,
    // pass `true` to clear transforms of children as well.
    box.ClearTransformsAfter(Time.Current);
}
```

- Note that `ClearTransformsAfter(time)` actually removes transforms starting at the given time, not after it. This was a mistake in naming and will be resolved soon.

### Applying transforms to the drawable from a different time before removing

Removing transforms will not revert them, the drawable will remain at its current state.

To revert the transform applied to the `Drawable`, or apply them from a given absolute clock time before removing, use `ApplyTransformsAt`.
```csharp
var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(1000);

// after 0.5 seconds, revert the transform and remove it from the box.
Scheduler.AddDelay(() => 
{
    // by default, this only applies to transforms applied to the box itself,
    // pass `true` to clear transforms of children as well.
    box.ApplyTransformsAt(Time.Current - 500);
    box.ClearTransforms();
}, 500);
```

## Rewinding support

// TODO

## Using custom easing functions

In cases the [default `Easing` functions](https://github.com/ppy/osu-framework/blob/fb3029668d12ef14dd43ed9ac71395765daf9efe/osu.Framework/Graphics/Easing.cs) aren't what you want for some transforms, you can define your own `IEasingFunction` and pass it to the transforms:

```csharp
public readonly struct SpecialEasingFunction : IEasingFunction
{
    public double ApplyEasing(double time) => time;
}

var box = new Box { Size = new Vector2(50) }

Add(box); // the target of transforms must always be in the draw hierarchy and loaded before operating on it.

box.FadeInFromZero(1000, new SpecialEasingFunction());
```

- `IEasingFunction.ApplyEasing` accepts a time value in the range [0-1], and returns an eased value of it. For examples, you can refer to the [`DefaultEasingFunction`](https://github.com/ppy/osu-framework/blob/d284856a6a7a341ab12f1f1169cac30f4aec8caa/osu.Framework/Graphics/Transforms/DefaultEasingFunction.cs) implementation.

## Best practices

Some basic things to note:

- Transforming before a drawable is loaded (ie. `LoadComplete`) will cause the transforms to play out immediately. This is due to the drawable not yet having a clock to work with. If you must queue transforms before load, make sure to use a `Schedule(() => {})` call.
- Transforms are not free and should not be run every frame. Please use interpolation for such cases.
- Generally, we recommend not operating on oneself with transforms (ie. `this.FadeIn()`). This is due to the potential of conflicts between internal and external calls, which could overwrite or cause unexpected behaviour. Create a private nested container and operate on that instead.

