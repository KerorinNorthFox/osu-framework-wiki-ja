# Handling Input

Once you have completed [setting up your first project](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project), we are ready to start building our game! In addition to having the ability to [handle input events](https://github.com/ppy/osu-framework/wiki/Handling-input-events) of various types, you can also set up a [KeyBindingContainer](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/Bindings/KeyBindingContainer.cs) which allows you to name actions in an enumerator, as well as choose which key handling mode should be used in the container.

* [Creating a new KeyBindingContainer](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#creating-a-new-keybindingcontainer)
  * [Create an enum for our custom actions](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#create-an-enum-for-our-custom-actions)
  * [Create a KeyBindingContainer class](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#create-a-keybindingcontainer-class)
  * [Create default keybindings for our KeyBindingContainer](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#create-default-keybindings-for-our-keybindingcontainer)
  * [Key Binding Modes](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#key-binding-modes)
* [Handling key bound actions](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#handling-key-bound-actions)
  * [Example](https://github.com/ppy/osu-framework/wiki/Setting-up-keybinding-containers#example)

## Creating a new KeyBindingContainer
### Create an enum for our custom actions
First, we should name every input we want to be a part of this KeyBindingContainer. This lets us use the type field when inheriting the KeyBindingContainer<T> class so that it knows what action data it should map input actions to.

For this example, we will map our W, A, and S keys to our Jump, Left, and Right respectively. We will create our enum type like so:
```csharp
public enum InputAction
{    
    Left,
    Right,
    Jump
}
```
It is recommended you create this new type inside the same file where you're creating the new KeyBindingContainer itself in step 2. That way, it will be easier to keep track of where it is if someone were to try to find them, as well as guarantee that both exist in the same namespace.
### Create a KeyBindingContainer class
We will then take the enum type we just created and use it to create our KeyBindingContainer. 
```csharp
public class AwesomeKeyBindingContainer : KeyBindingContainer<InputAction>
{
}
```
### Create default keybindings for our KeyBindingContainer
We can now map key presses to the action we wish to perform! To do so, we should first create a default set of keybinds. In our newly created KeyBindingContainer, override the existing DefaultKeyBindings property with your own:
```csharp
public class AwesomeKeyBindingContainer : KeyBindingContainer<InputAction>
{
  public override IEnumerable<KeyBinding> DefaultKeyBindings => new[]
  {
     new KeyBinding(new[] { InputKey.A }, InputAction.Left),
     new KeyBinding(new[] { InputKey.W }, InputAction.Jump),
     new KeyBinding(new[] { InputKey.D }, InputAction.Right)
  }
}
```

### Key Binding Modes

There are two types of There are two modes that can be specified on the creation of a new keybinding container via the parameters. One is the [SimultaneousBindingMode](https://github.com/ppy/osu-framework/blob/e143142928ebf87a10777217ecf0d6dc45082282/osu.Framework/Input/Bindings/KeyBindingContainer.cs#L285), and the other is the [KeyCombinationMatchingMode](https://github.com/ppy/osu-framework/blob/e143142928ebf87a10777217ecf0d6dc45082282/osu.Framework/Input/Bindings/KeyCombination.cs#L316). The SimultaneousBindingMode specifies whether or not an input should be valid based on whether or not other buttons are pressed simultaneously, whereas the KeyCombinationMatchingMode specifies how strict the keybinding should be in regards to whether or not the exact buttons are pressed. 

For a better visual representation of these modes and how they restrict multiple key press events, please see [TestCaseKeyBindings](https://github.com/ppy/osu-framework/blob/master/osu.Framework.Tests/Visual/TestCaseInput/TestCaseKeyBindings.cs).

## Handling key bound actions

Now that we have created our KeyBindingContainer, we can add behavior that triggers when the key-binded action has been fired. The [IKeyBindingHandler<T>](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/Bindings/IKeyBindingHandler.cs) interface provides a way for bound input events to be handled by any class.

For example, to create a player class that inherits RigidBodyContainer and implements IKeybindingHandler, we would do this:

```csharp
public class Player : RigidBodyContainer<Drawable>, IKeyBindingHandler<InputAction>
{
}
```

The IKeyBindingHandler interface provides access to the following:

* OnPressed(action) is triggered when any action is pressed. The action type depends on the type specified in the type specifier when you inherit IKeyBindingHandler<T>. Return true if this action should block the action from traversing the scene graph up to its parent.
* OnReleased(action) is triggered when any action is released. Likewise, return true if this action should be blocked from traversing the scene graph for handling for the Drawable's parents.

### Example
Our completed player RigidBodyContainer with keybinding handling would then look something like this:

```csharp
public class Player : RigidBodyContainer<Drawable>, IKeyBindingHandler<InputAction>
{
    private float constantXForce;
        
    [BackgroundDependencyLoader]
    private void load(TextureStore textureStore)
    {
        Child = new Box
        {
            Anchor = Anchor.Centre,
            Origin = Anchor.Centre,
            Size = new Vector2(150, 150),
        };
        Position = new Vector2(500, 500);
        Size = new Vector2(200, 200);
        Rotation = 45;
        Masking = true;
    }

    private const float PLAYER_VELOCITY = 500f;

    public bool OnPressed(InputAction action)
    {
        switch (action)
        {
            case InputAction.Jump:
                Velocity = new Vector2(constantXForce, Velocity.Y - 500);
                break;
            case InputAction.Right:
                constantXForce += PLAYER_VELOCITY;
                break;
            case InputAction.Left:
                constantXForce -= PLAYER_VELOCITY;
                break;
        }
        return true;
    }
        
    public bool OnReleased(InputAction action)
    {
        switch (action)
        {
            case InputAction.Left:
                constantXForce += PLAYER_VELOCITY;
                break;
            case InputAction.Right:
                constantXForce -= PLAYER_VELOCITY;
                break;
        }
        return true;
    }
        
    protected override void Update()
    {
        base.Update();   
        Velocity = new Vector2(constantXForce, Velocity.Y);
    }
}
```