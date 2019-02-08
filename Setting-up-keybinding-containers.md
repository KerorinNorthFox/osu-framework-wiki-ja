# Handling Input

Once you have completed [setting up your first project](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project), we are ready to start building our game! In addition to having the ability to [handle input events](https://github.com/ppy/osu-framework/wiki/Handling-input-events) of various types, you can also set up a [KeyBindingContainer](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/Bindings/KeyBindingContainer.cs) which allows you to name actions in an enumerator, as well as choose which key handling mode should be used in the container. 

## Creating a new KeyBindingContainer
### Create an enum for our custom actions
First, we should name every input we want to be a part of this KeyBindingContainer. This lets us use the type field when inheriting the KeyBindingContainer<T> class so that it knows what action data it should map input actions to.

For this example, we will map our W, A, S, and D keys to our Up, Left, Right, and Down actions respectively. We will create our enum type like so:
```
public enum InputAction
{    
    Left,
    Right,
    Down,
    Up
}
```
It is recommended you create this new type inside the same file where you're creating the new KeyBindingContainer itself in step 2. That way, it will be easier to keep track of where it is if someone were to try to find them, as well as guarantee that both exist in the same namespace.
### Create a KeyBindingContainer class
We will then take the enum type we just created and use it to create our KeyBindingContainer. 
```
public class AwesomeKeyBindingContainer : KeyBindingContainer<InputAction>
{
}
```
### Create default keybindings for our KeyBindingContainer
We can now map key presses to the action we wish to perform! To do so, we should first create a default set of keybinds. In our newly created KeyBindingContainer, override the existing DefaultKeyBindings property with your own:
```
public class AwesomeKeyBindingContainer : KeyBindingContainer<InputAction>
{
   public override IEnumerable<KeyBinding> DefaultKeyBindings => new[]
   {
       new KeyBinding(new[] { InputKey.A }, InputAction.Left),
       new KeyBinding(new[] { InputKey.W }, InputAction.Up),
       new KeyBinding(new[] { InputKey.D }, InputAction.Right),
       new KeyBinding(new[] { InputKey.S }, InputAction.Down),
   }
}
```

### Key Binding Modes

There are two types of There are two modes that can be specified on the creation of a new keybinding container via the parameters. One is the [SimultaneousBindingMode](https://github.com/ppy/osu-framework/blob/e143142928ebf87a10777217ecf0d6dc45082282/osu.Framework/Input/Bindings/KeyBindingContainer.cs#L285), and the other is the [KeyCombinationMatchingMode](https://github.com/ppy/osu-framework/blob/e143142928ebf87a10777217ecf0d6dc45082282/osu.Framework/Input/Bindings/KeyCombination.cs#L316). The SimultaneousBindingMode specifies whether or not an input should be valid based on whether or not other buttons are pressed simultaneously, whereas the KeyCombinationMatchingMode specifies how strict the keybinding should be in regards to whether or not the exact buttons are pressed. 

For a better visual representation of these modes and how they restrict multiple key press events, please see [TestCaseKeyBindings](https://github.com/ppy/osu-framework/blob/master/osu.Framework.Tests/Visual/TestCaseInput/TestCaseKeyBindings.cs).

## Handling key binding actions

Now that we have created our KeyBindingContainer, we can now add behavior that triggers when the key-binded action has been fired. 

### 