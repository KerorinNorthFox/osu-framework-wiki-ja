All initial development of (and further improvements applied) to components in a project should be developed in the VisualTests environment. There are several advantages to doing this.

# Hot Reload

Test scenes are integrated with the [.NET Hot Reload](https://devblogs.microsoft.com/dotnet/introducing-net-hot-reload/) mechanism. Here is an example of how this looks in practice:

https://github.com/ppy/osu-framework/assets/20418176/a3598d28-5347-45a2-9ef7-43fab5895a83

Keep in mind that the Hot Reload facility is subject to some limitations and some changes will require a full application restart. For more information, see [*Supported Edits in Edit & Continue (EnC)*](https://github.com/dotnet/roslyn/blob/2a9938ad11e432fdd274353f57083d4f6f83edf8/docs/wiki/EnC-Supported-Edits.md).

# Steps and automated testing

You can use the `AddStep` function to add automatic steps which should be performed. Asserts can also be added as steps. These steps can be run in an interactive environment by the user, but also run in a headless execution, allowing for CI process testing.

The types of steps available to be added are as follows: 

* [`AddStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L290) creates a step that runs a method. Completes successfully if no exceptions are caught.
* [`AddRepeatStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L322) creates a step that runs a method a specified amount of times. Completes successfully if no exceptions are caught.
* [`AddToggleStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L330) toggles a flag.
* [`AddWaitStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L346) adds a step that waits a specified amount of time before continuing to the next step.
* [`AddSliderStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L354) adds a step that creates a slider-bar that adjusts a set value.
* [`AddAssert()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L362) creates a step that fails if the specified value does not return true.
* [`AddUntilStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L338) adds a step that attempts to run until a condition becomes true, or fails when it times out.

Both `AddAssert()` and `AddUntilStep()` support the [NUnit assertion constraint model](https://docs.nunit.org/articles/nunit/writing-tests/assertions/assertion-models/constraint.html), i.e.:

```csharp
AddAssert("no text is selected", () => textBox.SelectedText, () => Is.Empty);
```

Using this style is highly recommended as the NUnit matchers provide vastly improved debugging output in case of failure compared to just plain true/false assertions.

# Caveats

* Hot reload has no effect if the application is started with a debugger attached. Attempting to attach debugger after launching and applying changes via hot reload will fail with error `0x80131c69` (`CORDBG_E_ASSEMBLY_UPDATES_APPLIED`).
* If the debugger is attached, `UntilStep`s will never time out. This is a convenience feature to improve debugging of the failing condition in the until step.