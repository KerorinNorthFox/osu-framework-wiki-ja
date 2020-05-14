All initial development of (and further improvements applied) to components in a project should be developed in the VisualTests environment. There are several advantages to doing this.

# Dynamic compilation

TestScenes support a limited dynamic compilation engine. This allows for automatic compilation when a `.cs` file is changed, followed by reloading of the TestScene. Keep in mind a few limitations:

- This only works with editors that do atomic file writes (ie. if using Rider, you need to [turn off safe file writes](https://puu.sh/yr5bk/43a073a194.png)).
- You must have the TestScene you are interested in currently displayed before making file changes.
- The first time a dynamic compilation is initialised will take a while to process the dependency graph. Further compilations will be almost instant.

Here is an example of how this looks in practice:

![](https://puu.sh/yr5sB/9a2b547dcb.gif)

# Steps and automated testing

You can use the `AddStep` function to add automatic steps which should be performed. Asserts can also be added as steps. These steps can be run in an interactive environment by the user, but also run in a headless execution, allowing for CI process testing.

The types of steps available to be added are as follows: 

* [AddStep](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L290) creates a step that runs a method. Completes successfully if no exceptions are caught.
* [AddRepeatStep](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L322) creates a step that runs a method a specified amount of times. Completes successfully if no exceptions are caught.
* [AddToggleStep](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L330) toggles a flag.
* [AddUntilStep](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L338) adds a step that attempts to run until a condition becomes true, or fails when it times out.
* [AddWaitStep](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L346) adds a step that waits a specified amount of time before continuing to the next step.
* [AddSliderStep](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L354) adds a step that creates a slider-bar that adjusts a set value.
* [AddAssert](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L362) creates a step that fails if the specified value does not return true.
