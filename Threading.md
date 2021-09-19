With regard to threading, osu!framework games can run in two modes:

* In _multi-threaded_ execution mode, the game is ran on four main threads:

    1. The input thread:
        - usually runs at 1000Hz
        - interfaces with the windowing toolkit used (SDL2 by default, osuTK also available)
        - responsible for handling input events and forwarding them to components/drawables
    2. The audio thread:
        - usually runs at 1000Hz
        - interfaces with the audio subsystem of the OS via the BASS audio library
    3. The update thread:
        - usually runs at twice the framerate limit (which can be toggled using the `FrameSync` framework setting, or the <kbd>Ctrl</kbd>+<kbd>F7</kbd> shortcut)
        - is responsible for running update code (`UpdateSubTree()`, `Update()`) for all components in the game's drawable hierarchy
    4. The draw thread:
        - runs at the framerate limit specified
        - dispatches draw calls to the GPU (currently using OpenGL, more backends planned in the future)

* In _single-thread_ execution mode, the four threads above run on one OS thread in an interleaved fashion.

The threading mode can be set using the `ExecutionMode` framework setting, or using the <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F7</kbd> shortcut.

Note that in either mode, it is possible that other tasks, such as asynchronous loads of drawables via `CompositeDrawable.LoadComponentAsync()` (see [docs on asynchronous loading](https://github.com/ppy/osu-framework/wiki/Asynchronous-loading)), or `Task`s running on the .NET runtime's thread pool, can use additional OS threads.

# Scheduling

Relatively often, there is a need to change the state of a `Drawable` or `Component` in response from a callback which is not executing on the game's update thread.
An example of this would be updating the text and transforms to a `SpriteText` in response to a web request.
To ensure that such an operation can be executed in a safe manner, each `Drawable` or `Component` has an internal `Scheduler`, that can collect such pending operations and execute them at the correct time:

```csharp
request = CreateRequest();
request.Failure += exception => Schedule(() =>
{
    errorMessageSpriteText.Text = exception.Message;
    errorMessageSpriteText.FadeTo(Colour4.Red).Then().FadeTo(Colour4.White, 1000);
});
```

While the above use case is to be considered the primary one for schedulers, they can also be used to handle a variety of other tasks, as well:

* If a drawable - for whatever reason - is not being actively updated (due to being not present, masked away, not loaded yet), scheduling a task onto it ensures that the task may execute on the nearest possible occasion when the drawable is back in the active scene graph.
* The return value of a `Schedule()` call is a `ScheduledDelegate`, which can be later cancelled. This can be used to enqueue operations to be executed at some point in the future, but still be able to early-cancel them if some other condition is met.
* Aside from `Schedule()`, the `Scheduler` itself is also exposed as protected on each drawable, and provides additional capabilities, such as:

    - `Scheduler.AddOnce(task)` can be used to ensure that an operation runs at most once per frame.
      This is most often useful when called from an input event handler (whether mouse, keyboard, or any other peripheral), to avoid performing needless updates.

    - `Scheduler.AddDelayed(task, timeUntilRun)` can be used to run an operation at a set time in the future, relative to now.
      That method also has a `repeat` argument that can be used for recurring operations (they will execute regularly every `timeUntilRun` milliseconds).

* As a last-ditch attempt, schedulers can sometimes also be used to fix various one-frame issues, such as auto-size containers flickering when their contents are being changed.
  This sort of usage is however discouraged, as it can cause headaches down the line due to complicating interactions between components.

In general schedulers are a powerful tool, but ones that should be used carefully, as they change the usual flow of operations and therefore delay their effects.
Nesting schedules, or having chains of scheduled operations will complicate debugging significantly.

# Asynchronous disposal

All drawables/components in a game, upon being removed from their parent `CompositeDrawable`s, are enqueued onto a game-global async disposal queue.
The queue uses TPL threadpool threads to run each of the drawables' `Dispose()` methods.

To suppress this behaviour, methods such as `CompositeDrawable.Remove()` or `CompositeDrawable.Clear(false)` can be used to remove a drawable from the hierarchy without calling its `Dispose()` implicitly.

# Setting thread culture

To change all of the main game threads to a new culture, you can set the `Locale` framework setting to the name of the desired locale, which should correspond to the value of `CultureInfo.Name` of the desired culture.
Doing this will change culture on all of the main threads, as well as set the default culture for any new threads.
Some existing threads not managed by the framework may still continue operating using the old culture, however.