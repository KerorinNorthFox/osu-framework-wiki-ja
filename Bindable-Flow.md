`osu-framework` utilizes `Bindable<T>` objects to distribute data between components. They provide functionality to automatically remove communication between `Bindable<T>` objects when finalized, serving as a safer alternative to C#'s `event`.

Creating a `Bindable<T>`
========================

In `public`/`protected`/`internal` scenarios, it is recommended to store a private `Bindable<T>` backing and re-expose it publicly as one of the read-only interfaces - `IBindable` or `IBindable<T>`, depending on how much access the outside objects should have. 

In `private` scenarios, it is simplest to store bindables as `Bindable<T>`.

```
public class MyClass
{
    private readonly Bindable<int> privateBacking = new Bindable<int>();
    public IBindable<int> PublicBindable => privateBacking;

    private readonly Bindable<int> privateBindable = new Bindable<int>();
}
```

A few subclasses, such as `BindableDouble`, extend `Bindable<T>` to provide additional functionality such as range-limiting of numeric values. These are available under the `osu.Framework.Configuration` namespace.

Chaining `Bindable<T>`s together
===========================

`Bindable<T>` supports the ability to "bind" to other `Bindable<T>`s. When either one of the bindable's values changes, it will also set the value on the other.

This is available through the `Bindable<T>.BindTo()` method.

```
var x = new Bindable<int>(1);
var y = new Bindable<int>(2);

x.BindTo(y);
Assert.IsTrue(x.Value == 2); // BoundTo() immediately sets x's value to y's

y.Value = 10;
Assert.IsTrue(x.Value == 10); // The value set to y previously is propagated to x

x.Value = 5;
Assert.IsTrue(y.Value == 5); // Same as above - dual-way communication
```

Observing the values of a `Bindable<T>`
=======================================

The value of a `Bindable<T>` can be observed through the `ValueChanged` event. This returns the new value.

```
var x = new Bindable<int>();

x.ValueChanged += newValue => Assert.IsTrue(newValue == 2);
x.Value = 2;
```

In some scenarios it may be desirable to have the `ValueChanged` delegate invoked once after being set. It is possible to achieve this in two ways:

```
var x = new Bindable<int>();

// #1:
x.BindValueChanged(myValueFunc, true); // Recommended

// #2:
x.ValueChanged += myValueFunc;
x.TriggerChange();
```

Breaking the `Bindable<T>` chain
================================

The binding chain between `Bindable<T>`s is weak. That is, bound `Bindable<T>`s will not cause each other to forego garbage collection.

When a `Bindable<T>` is finalized, it will automatically unbind other `Bindable<T>`s bound via `BindTo()` and any subscribers to its `ValueChanged` event. This is subject to the garbage collector and may not occur instantly.

When a `Drawable` disposes, it forces all `Bindable<T>`s contained as `private`/`protected`/`public`/`internal` fields and properties to unbind. This generally occurs as soon as the `Drawable` is removed from the draw hierarchy via `Clear(true)`/`ClearInternal(true)` or setting `InternalChildren`/`Children`, or via the garbage collector when all references to the `Drawable` are lost.

It may be desired to unbind at an arbitrary point in time where garbage collection is not guaranteed by the above. A few methods are provided to achieve this:

* `UnbindEvents()` - Unbinds all subscribers bound via `ValueChanged`.
* `UnbindBindings()` - Unbinds all `Bindable<T>`s bound via `BoundTo()`.
* `UnbindAll()` - Combines `UnbindEvents()` and `UnbindBindings()`.