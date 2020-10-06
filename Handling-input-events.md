Input handling follows a topological model whereby the top-most (visually) `Drawable` handles input prior to anything else in the game. Hierarchically between a single parent-child relationship, the child handles input before the parent.

```
Parent              < Handler #4
    Child_1         < Handler #3
    Child_2         < Handler #2
        Child_2_1   < Handler #1
```

## Positional vs non-positional input

"Positional" input refers to any input that depends on a screen-space position (e.g. hover). "Non-positional" input refers to any other input (e.g. a button press).

## Input events

![input system](https://user-images.githubusercontent.com/1329837/43699274-188d8126-9989-11e8-9e39-a2079a4ea42b.png)

[source](https://www.lucidchart.com/documents/view/9bffbf73-0c99-4c38-b0f1-2f3f80c47a49/0)

## Individual event handlers
##### Positional
```csharp
protected virtual bool OnMouseMove(MouseMoveEvent e);

protected virtual bool OnHover(HoverEvent e);
protected virtual void OnHoverLost(HoverLostEvent e);

protected virtual bool OnMouseDown(MouseDownEvent e);
protected virtual void OnMouseUp(MouseUpEvent e);

protected virtual bool OnClick(ClickEvent e);
protected virtual bool OnDoubleClick(DoubleClickEvent e);

protected virtual bool OnDragStart(DragStartEvent e);
protected virtual void OnDrag(DragEvent e);
protected virtual void OnDragEnd(DragEndEvent e);

protected virtual bool OnScroll(ScrollEvent e);
```

##### Non-positional
```csharp
protected virtual bool OnKeyDown(KeyDownEvent e);
protected virtual void OnKeyUp(KeyUpEvent e);

protected virtual bool OnTouchDown(TouchDownEvent e);
protected virtual void OnTouchMove(TouchMoveEvent e);
protected virtual void OnTouchUp(TouchUpEvent e);

protected virtual bool OnJoystickPress(JoystickPressEvent e);
protected virtual void OnJoystickRelease(JoystickReleaseEvent e);
protected virtual bool OnJoystickAxisMove(JoystickAxisMoveEvent e);

protected virtual bool OnMidiDown(MidiDownEvent e);
protected virtual void OnMidiUp(MidiUpEvent e);
```

These event handlers should be used when singular events are to be handled.

The boolean return value indicates whether the event should continue being propagated to other `Drawable`s in the scene graph.  
**Example:** If a child did not want its parent to receive an `OnHover()` event, it should override `OnHover()` to return `true`.

`OnHoverLost()` is an exception which is unconditionally invoked on every un-hovered `Drawable`.

**Example:**
```csharp
protected override bool OnClick(ClickEvent e)
{
    this.ScaleTo(1.25f, 50).Then().ScaleTo(1f, 50);
    return true;
}
```

Input handlers that correspond to continuations/resolutions of previous input, such as `OnMouseUp`, `OnDragEnd`, `OnKeyUp`, etc., will only fire on drawables that registered to handle the original input by returning `true` - for the examples above, that would be `OnMouseDown`, `OnDragStart`, `OnKeyDown`, etc. Those events cannot be suppressed.

## Aggregate event handler
```csharp
protected virtual bool Handle(UIEvent e);
```

This event handler should be used when groups of events with identical implementations are to be handled.

This counts as both a positional and non-positional event handler and works well when combined with the `HandleNonPositionalInput` and `HandlePositionalInput` properties described below.

**Example:**
```csharp
protected override bool Handle(UIEvent e)
{
    switch (e)
    {
        case MouseEvent _:
            // Stop all mouse events from being handled by other Drawables
            return true;
        default:
            // Let other Drawables handle everything else
            return false;
    }
}
```

## Controlling whether input is to be handled

By default, any `Drawable` that implements the handlers described above will receive all events appropriate for the types of input listed.

```csharp
public virtual bool HandleNonPositionalInput;
public virtual bool HandlePositionalInput;
```

The above properties are provided to control whether a `Drawable` should receive the types of input events.  
**Example**: If a `Drawable` did not want to handle any click, drag, and hover events at some point, it could achieve this by overriding `HandlePositionalInput` to return `false`.

**Warning**: Overriding the above properties will cause the `Drawable` to be _considered_ for (but not necessarily receive) that type of input, and thus serves as an anti-optimisation if the intention is to make the `Drawable` not considered for that type of input.

If a parent should control whether itself or its children should be considered for a type of input at all, the following properties are provided to control the behaviour:

```csharp
public virtual bool PropagateNonPositionalInputSubTree;
public virtual bool PropagatePositionalInputSubTree;
```

**Example:**
```csharp
class MyContainer : FillFlowContainer
{
    public MyContainer()
    {
        for (int i = 0; i < 1024; i++)
            Add(new Button { Size = new Vector2(50) });
    }

    // Whoah! The buttons we added are just for show, they don't actually handle clicks!
    public override bool PropagatePositionalInputSubTree => false;
}
```

## Receiving and handling focus
```csharp
public virtual bool RequestsFocus;
public virtual bool AcceptsFocus;

protected virtual void OnFocus(FocusEvent e);
protected virtual void OnFocusLost(FocusLostEvent e);
```

Focus may be received via both positional and non-positional input. A `Drawable` with `AcceptsFocus = false` will never receive focus.

##### Positional
A `Drawable` receives focus when `OnClick()` returns `true`.

##### Non-positional
The top-most `Drawable` with `RequestsFocus = true` and `HandleNonPostionalInput = true` will receive focus if nothing else has focus.

The `OnFocusLost()` method is unconditionally invoked on the un-focused `Drawable`.

**Example:**
```csharp
class MyDrawable : CompositeDrawable
{
    public override bool AcceptsFocus => true;
    protected override bool OnClick(ClickEvent e) => true;

    protected override void OnFocus(FocusEvent e) => this.ScaleTo(1.5f, 50);
    protected override void OnFocusLost(FocusLostEvent e) => this.ScaleTo(1f, 50);
}
```