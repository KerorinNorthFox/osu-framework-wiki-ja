All initial development of (and further improvements applied) to components in a project should be developed in the VisualTests environment. There are several advantages to doing this.

# Dynamic compilation

TestCases support a limited dynamic compilation engine. This allows for automatic compilation when a `.cs` file is changed, followed by reloading of the TestCase. Keep in mind a few limitations:

- This only works with editors that do atomic file writes (ie. if using Rider, you need to [turn off safe file writes](https://puu.sh/yr5bk/43a073a194.png)).
- You must have the TestCase you are interested in currently displayed before making file changes.
- By default, changes are only tracked in the current TestCase and its target class. If you are in `TestCaseGraph`, you will be able to make changes in `TestCaseGraph.cs` and `Graph.cs`.

If you wish to make changes to more classes, for instance if your TestCase is testing multiple tightly coupled classes, please use the `RequiredTypes` property to specify the required classes. Note that you can do this in real-time as you are going to be adding the override in the TestCase itself.

```csharp
public override IReadOnlyList<Type> RequiredTypes => new[] { typeof(GraphBar), typeof(GraphContainer) };
```

Here is an example of how this looks in practice:

![](https://puu.sh/yr5sB/9a2b547dcb.gif)

# Steps and automated testing

You can use the `AddStep` function to add automatic steps which should be performed. Asserts can also be added as steps. These steps can be run in an interactive environment by the user, but also run in a headless execution, allowing for CI process testing.
