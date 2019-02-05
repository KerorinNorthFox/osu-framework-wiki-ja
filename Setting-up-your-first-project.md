# Setting up your first project

This tutorial is designed to get you started with a new project using osu-framework. It is not meant to be a guide for contribution to the osu-framework project.

## Targeting desktop platforms
You can find this code and more to help you get started in [`SampleGame`](https://github.com/ppy/osu-framework/tree/master/SampleGame) and [`SampleGame.Desktop`](https://github.com/ppy/osu-framework/tree/master/SampleGame.Desktop)

1. Create a new WinExe project. 

Simply set the ouput type in your .csproj file to `<OutputType>WinExe</OutputType>`, or change your existing output type to WinExe. If you're using Visual Studio or Rider, set the project type to "Windows Application"

2. Install the nuget package from https://www.nuget.org/packages/ppy.osu.Framework/.
3. Create a new game class that derives osu.Framework.Game.

The following code creates a cube with a rigid body container.
```
namespace AwesomeGame
{
    public class AwesomeGame : Game
    {
        RigidBodySimulation sim;
        
        [BackgroundDependencyLoader]
        private void load()
        {
            Child = sim = new RigidBodySimulation { RelativeSizeAxes = Axes.Both };
            
            RigidBodyContainer<Drawable> rbc = new RigidBodyContainer<Drawable>
            {
                Child = new Box
                {
                    Anchor = Anchor.Centre,
                    Origin = Anchor.Centre,
                    Size = new Vector2(150, 150),
                    Colour = Color4.Tomato,
                },
                Position = new Vector2(500, 500),
                Size = new Vector2(200, 200),
                Rotation =  45,
                Colour = Color4.Tomato,
                Masking = true,
            };
            
            sim.Add(rbc);            
        }
    }
}
```
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

### Further reading
For more information regarding dependency injection via the BackgroundDependencyLoader attribute, please read [Dependency Injection](https://github.com/ppy/osu-framework/wiki/Dependency-Injection)

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

Note that in order for you to be able to run the visual tests, you will have to switch your game instance to the visual tests. To do that, you can either create your visual tests in a separate project and run them as a build configuration, or simply apply build configuration pre-processor checks in your entry point to run the appropriate game.

For example, if I have the VisualTests build configuration, rider will automatically create the pre-processor flag VISUALTESTS, which allows us to use `#if VISUALTESTS` to check the current build configuration.

### Adding tests to the test browser

Now that our test browser is discovering tests from the specified namespace, we can start adding tests! To do so, create a new class that derives [TestCase](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Testing/TestCase.cs) with the [TestFixture](http://nunit.org/docs/2.6/testFixture.html) attribute. From here, we can add steps to this test of various types:
* [AddStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L280) creates a step that runs a method. Completes successfully if no exceptions are caught.
* [AddRepeatStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L293) creates a step that runs a method a specified amount of times. Completes successfully if no exceptions are caught.
* [AddToggleStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L301) toggles a flag.
* [AddUntilStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L309) adds a step that attempts to run until a condition becomes true, or fails when it times out.
* [AddWaitStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L317) adds a step that waits a specified amount of time before continuing to the next step.
* [AddSliderStep](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L325) adds a step that creates a slider-bar that adjusts a set value.
* [AddAssert](https://github.com/ppy/osu-framework/blob/d2d47c58585e6ceb8fcf4d296bc4a993753c2a1d/osu.Framework/Testing/TestCase.cs#L333) creates a step that fails if the specified value does not return true.

### Example Test
The following code adds a simple cube to the visual test browser that we created above. The cube has a rigid body attached, and should drop to the bottom of the screen when created. From here, we can choose to check the behavior of the cube by asserting that the cube eventually reaches the bottom via AddAssert.

```
namespace AwesomeGame.VisualTests
{
    [TestFixture]
    public class RigidCubeTest : TestCase
    {
        private RigidBodySimulation sim;

        [Test]
        public void AwesomeTestName()
        {
            AddStep("Drop a cube", DropCube);
        }

        private void DropCube()
        {
            Child = sim = new RigidBodySimulation { RelativeSizeAxes = Axes.Both };
            
            RigidBodyContainer<Drawable> rbc = new RigidBodyContainer<Drawable>
            {
                Child = new Box
                {
                    Anchor = Anchor.Centre,
                    Origin = Anchor.Centre,
                    Size = new Vector2(150, 150),
                    Colour = Color4.Tomato,
                },
                Position = new Vector2(500, 500),
                Size = new Vector2(200, 200),
                Rotation =  45,
                Colour = Color4.Tomato,
                Masking = true,
            };
            
            sim.Add(rbc);
        }
    }
}
```

### Test setup
The [Setup](https://nunit.org/docs/2.2/setup.html) NUnit attribute marks a method as a setup method that runs as a step before every group of tests in a test method. The steps created by this attribute get added to the visual test browser as well.

### Further reading
For additional reading on visual tests, please refer to [Dynamic Compilation and Visual Testing](https://github.com/ppy/osu-framework/wiki/Development-and-Testing)