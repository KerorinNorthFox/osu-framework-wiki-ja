# Screens and Screen Stacks
osu!framework utilizes and implements a [`Screen`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Screens/IScreen.cs) concept. Screens let us specify single "views" that can be stacked on with other screens, or exited to reveal screens stacked underneath. 

Visually, imagine stacking one sheet of paper on top of another. The forward facing sheet is shown to the user, while other screens remain underneath it ready to be resumed.

Only one screen can be active at a time within a single screen stack. However, multiple different screen stacks may exist to layer screens on top of one another. 

* [Creating a new screen stack](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#creating-a-new-screen-stack)
* [Creating a new screen](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#creating-a-new-screen)
* [Handling screen transitions](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#handling-screen-transitions)
 * [Example](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#example)

## Creating a screen stack
Simply add a new instance of a screen stack into your draw hierarchy.

```csharp
private ScreenStack awesomeScreenStack

[BackgroundDependencyLoader]
private void load()
{
    Add(awesomeScreenStack = new ScreenStack());
}
```

## Creating a new screen
When a screen stack is first initialized, it will be empty. We will need to create a screen before it has anything that can show the user. 

Screens can be created by adding Drawables to it like any other CompositeDrawable. 

```csharp
public class AwesomeScreen : Screen
{
    public AwesomeScreen()
    {
        AddInternal(new Box
        {
            Colour = Color4.Tomato,
            Origin = Anchor.Centre,
            Anchor = Anchor.Centre
        });
    }
}
```

If this screen is the first screen you are pushing to the stack, you should push it to the stack directly using `ScreenStack.Push()`. If you already have a screen in the stack, you can also push to the screen using `Screen.Push()`.

## Handling screen transitions
Screens contain the following methods that are invoked whenever a screen change involving it would occur:
* OnEntering - Invoked when the screen is added to the top of the stack using `Screen.Push()`
* OnExiting - Invoked when the screen is exited for the screen under it using `Screen.Exit()`
* OnResuming - Invoked when the screen is made the active screen as a result of the current screen being exited.
* OnSuspending - Invoked when another screen is pushed to the top of the stack, replacing this one.

### Example
We can take care of inwards screen related transitions inside the screen we're transitioning into using the OnEntering and OnResuming methods. The FadeInFromZero method is great for fading in a new screen, where as FadeIn would let us adjust the fade from any alpha the screen may already be at.
```csharp
public OnEntering(IScreen next)
{
    FadeInFromZero<Screen>(500, Easing.OutQuint);
}

public OnSuspending(IScreen next)
{
    FadeIn(500, Easing.OutQuint);
}
```