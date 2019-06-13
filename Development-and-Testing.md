All initial development of (and further improvements applied) to components in a project should be developed in the VisualTests environment. There are several advantages to doing this.

# Dynamic compilation

TestScenes support a limited dynamic compilation engine. This allows for automatic compilation when a `.cs` file is changed, followed by reloading of the TestScene. Keep in mind a few limitations:

- This only works with editors that do atomic file writes (ie. if using Rider, you need to [turn off safe file writes](https://puu.sh/yr5bk/43a073a194.png)).
- You must have the TestScene you are interested in currently displayed before making file changes.
- By default, changes are only tracked in the current TestScene and its target class. If you are in `TestSceneGraph`, you will be able to make changes in `TestSceneGraph.cs` and `Graph.cs`.

If you wish to make changes to more classes, for instance if your TestScene is testing multiple tightly coupled classes, please use the `RequiredTypes` property to specify the required classes. Note that you can do this in real-time as you are going to be adding the override in the TestScene itself.

```csharp
public override IReadOnlyList<Type> RequiredTypes => new[] { typeof(GraphBar), typeof(GraphContainer) };
```

Here is an example of how this looks in practice:

![](https://puu.sh/yr5sB/9a2b547dcb.gif)

# Steps and automated testing

You can use the `AddStep` function to add automatic steps which should be performed. Asserts can also be added as steps. These steps can be run in an interactive environment by the user, but also run in a headless execution, allowing for CI process testing.

The types of steps available to be added are as follows: 

* [AddStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L280) creates a step that runs a method. Completes successfully if no exceptions are caught.
* [AddRepeatStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L293) creates a step that runs a method a specified amount of times. Completes successfully if no exceptions are caught.
* [AddToggleStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L301) toggles a flag.
* [AddUntilStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L309) adds a step that attempts to run until a condition becomes true, or fails when it times out.
* [AddWaitStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L317) adds a step that waits a specified amount of time before continuing to the next step.
* [AddSliderStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L325) adds a step that creates a slider-bar that adjusts a set value.
* [AddAssert](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestScene.cs#L333) creates a step that fails if the specified value does not return true.
