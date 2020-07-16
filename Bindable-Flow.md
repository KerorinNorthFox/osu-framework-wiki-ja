`osu-framework` utilizes `Bindable<T>` objects to distribute data between components. In conjuncation with Drawable components, they provide functionality to automatically remove communication between `Bindable<T>` objects when finalized, serving as a safer alternative to C#'s `event`.

## Creating a `Bindable<T>`

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

## Binding `Bindable<T>`s together

`Bindable<T>` supports the ability to "bind" to other `Bindable<T>`s. When either one of the bindable's values changes, it will also set the value on the other.

This is available through the `.BindTo()` and `.GetBoundCopy()` methods.

```csharp
var x = new Bindable<int>(1);
var y = new Bindable<int>(2);

x.BindTo(y);
Assert.IsTrue(x.Value == 2); // BoundTo() immediately sets x's value to y's

y.Value = 10;
Assert.IsTrue(x.Value == 10); // The value set to y previously is propagated to x

x.Value = 5;
Assert.IsTrue(y.Value == 5); // Same as above - dual-way communication
```

```csharp
var x = new Bindable<int>(1);
var y = x.GetBoundCopy();

x.BindTo(y);
Assert.IsTrue(x.Value == 2); // BoundTo() immediately sets x's value to y's

y.Value = 10;
Assert.IsTrue(x.Value == 10); // The value set to y previously is propagated to x

x.Value = 5;
Assert.IsTrue(y.Value == 5); // Same as above - dual-way communication
```

Generally, when interacting with a bindable exposed by another class, one should always make a local bindable to track changes. **Binding to another class instance's bindable events should be avoided**, as it bypasses the reference safety provided by bindables.

## Observing the values of a `Bindable<T>`

The value of a `Bindable<T>` can be observed through the `ValueChanged` event. This returns the new value.

```
var x = new Bindable<int>();

x.ValueChanged += val => Assert.IsTrue(val.NewValue == 2);
x.Value = 2;
```

In some scenarios it may be desirable to have the `ValueChanged` delegate invoked once after being set up. It is possible to achieve this in two ways:

```
var x = new Bindable<int>();

// #1:
x.BindValueChanged(myValueFunc, true); // Recommended

// #2:
x.ValueChanged += myValueFunc;
x.TriggerChange();
```

## Breaking the `Bindable<T>` chain

The binding chain between `Bindable<T>`s is weak. That is, bound `Bindable<T>`s will not cause each other to forego garbage collection.

When a `Bindable<T>` is finalized, it will automatically unbind other `Bindable<T>`s bound via `BindTo()` and any subscribers to its `ValueChanged` event. This is subject to the garbage collector and may not occur instantly.

When a `Drawable` disposes, it forces all `Bindable<T>`s contained as `private`/`protected`/`public`/`internal` fields and properties to unbind. This generally occurs as soon as the `Drawable` is removed from the draw hierarchy via `Clear(true)`/`ClearInternal(true)` or setting `InternalChildren`/`Children`, or via the garbage collector when all references to the `Drawable` are lost.

It may be desired to unbind at an arbitrary point in time when finalization is not guaranteed by the above. A few methods are provided to achieve this:

* `UnbindEvents()` - Unbinds all subscribers bound via `ValueChanged`.
* `UnbindBindings()` - Unbinds all `Bindable<T>`s bound via `BoundTo()`.
* `UnbindAll()` - Combines `UnbindEvents()` and `UnbindBindings()`.

## Leasing

There may be a scenario where you want to restrict updates to a high level bindable, but still allow changes by a certain component (or subset of components). Leasing exists to fulfil this goal. While a bindable is leased, it will be set to a `Disabled` state, but the special `LeasedBindable` will bypass this check and allow the bindable's value to be changed (and propagate as usual).

```csharp
var x = new Bindable<int>();
var leased = x.BeginLease(false);

// x.Value == 0
// leased.Value == 0

// x.Disabled == true
// leased.Disabled == true

x.Value = 1; // invalid, will throw an exception
leased.Value = 1; // valid, even though disabled

// x.Value == 1
// leased.Value == 1

leased.Return();
```

Another use case for leasing is to take advantage of the reverting of value on `Return()`. This allows a subsystem to gain control over a bindable, make changes, and after that subsystem is done, return the original bindable to its original state.

```csharp
var x = new Bindable<int>(2);
var leased = x.BeginLease(true);

leased.Value = 1;

// x.Value == 1
leased.Return();
// x.Value == 2
```

Note that during a lease, the `Disabled` state may be changed from the `LeasedBindable`. It will be restored to the value before the lease began on `Return()`.