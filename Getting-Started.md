# Getting Started

This tutorial is designed to get you started with a new project using osu-framework. 

## Desktop
You can find this code and more to help you get started in [`SampleGame`](https://github.com/ppy/osu-framework/tree/master/SampleGame) and [`SampleGame.Desktop`](https://github.com/ppy/osu-framework/tree/master/SampleGame.Desktop)

1. Create a new WinExe project. 

Simply set the ouput type in your .csproj file to `<OutputType>WinExe</OutputType>`, or change your existing output type to WinExe. If you're using Visual Studio or Rider, set the project type to "Windows Application"

2. Install the nuget package from https://www.nuget.org/packages/ppy.osu.Framework/.
3. Create a new game class that derives osu.Framework.Game.
4. In your main() method, create a new instance of your game.
```
[STAThread]
public static void Main()
{
   using (Game game = new AwesomeGame())
   using (GameHost host = Host.GetSuitableHost(@"awesome-game"))
      host.Run(game);
}
```
### BackgroundDependencyLoader Attribute
The [BackgroundDependencyLoader](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Allocation/BackgroundDependencyLoaderAttribute.cs) attribute denotes a method to be the load method of a [Drawable](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/Drawable.cs). You can specify a type in the method parameters to attempt to grab an object of that type that has been [cached](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Caching/Cached.cs).

## Testing
### Setting up a test browser
osu-framework has a visual testing framework that is meant to help provide tests that can be verified both visually and systematically via [NUnit](https://nunit.org/). To start using visual tests, you will need to create a [TestBrowser](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Testing/TestBrowser.cs) and add it to your screen. In order for your test browser to discover tests, you will need to specify a namespace in which for the browser to look in when constructing it.
```
namespace AwesomeGame.VisualTests
{
    public class AwesomeGameTestRunner : AwesomeGame
    {
         [BackgroundDependencyLoader]
         private void load()
         {
             Child = new DrawSizePreservingFillContainer
             {
                 Children = new Drawable[]
                 {
                     new TestBrowser("AwesomeGame"), //Specify the namespace to discover tests from
                     new CursorContainer(),
                 },
             };
         }

         public override void SetHost(GameHost host)
         {
             base.SetHost(host);
             host.Window.CursorState |= CursorState.Hidden;
         }
    }
}
```

### Adding tests to the test browser

Now that our test browser is discovering tests from the specified namespace, we can start adding tests! To do so, create a new class that derives [TestCase](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Testing/TestCase.cs) with the [TestFixture](http://nunit.org/docs/2.6/testFixture.html) attribute. From here, we can add steps to this test of various types:
* [AddStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L280) creates a step that runs a method. Completes successfully if no exceptions are caught.
* [AddRepeatStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L293) creates a step that runs a method a specified amount of times. Completes successfully if no exceptions are caught.
* [AddToggleStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L301) toggles a flag.
* [AddUntilStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L309) adds a step that attempts to run until a condition becomes true, or fails when it times out.
* [AddWaitStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L317) adds a step that waits a specified amount of time before continuing to the next step.
* [AddSliderStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L325) adds a step that creates a slider-bar that adjusts a set value.
* [AddAssert](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L333) creates a step that fails if the specified value does not return true.

### Test setup
The [Setup] NUnit attribute marks a method as a setup method that runs as a step before every group of tests in a test method. The steps created by this attribute get added to the visual test browser as well.