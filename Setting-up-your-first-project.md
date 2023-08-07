This tutorial will show you how to create a new project integrating osu-framework from scratch. It is not meant to be a guide for contributing to the osu-framework project itself.

* [Getting started](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#getting-started)
  * [Create a new project](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#creating-a-new-project)
  * [Project structure](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#project-structure)
  * [Game logic](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#game-logic)
  * [Platform specific logic](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#platform-specific-logic)
* [Testing](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#testing)
  * [The Test Browser](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#the-test-browser)
  * [Adding tests to the test browser](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#adding-tests-to-the-test-browser)
  * [Example test](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#example-test)
* [Appendix](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#appendix)
  * [BackgroundDependencyLoader attribute](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#BackgroundDependencyLoader-attribute)
  * [Setup Attribute](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#setup-attribute)
  * [Further Reading](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project#further-reading)

# Getting started

## Creating a new project

To get started, a custom osu! game base project has been made available as a template pack in the .NET command line interface. Using this template, it is possible to quickly and easily generate a new project using `osu-framework` as a starting point for creating your own game.

1. In your command line utility of choice, navigate to the folder in which you want to create the project.
2. Make sure the `osu-framework`  project template is downloaded by running `dotnet new install ppy.osu.Framework.Templates`.
3. Run `dotnet new osu-framework-game -n <MyNewProjectName>` to generate a new folder containing the project. 

   > **Warning**
   > Do not use spaces or hyphens in your project name. This does not play nice with the templating system.

4. Open the folder, and open `MyNewProjectName.sln` to begin!

> In addition to the `osu-framework-game` template, a template named `osu-framework-flappy-game` is also available which serves as an example of creating a basic, but feature complete game with `osu-framework`.

## Project structure

When you open the solution file, you'll notice that there are three build projects inside it:

* `MyNewProject.Game`: The main project that integrates with `osu-framework` and allows you to extend it by adding your own classes. All of your game specific logic should be implemented in this project.
* `MyNewProject.Desktop`: As opposed to mobile devices, the `Desktop` project provides all of the system mechanisms and resources necessary to successfully run the project on desktop platforms. Separating this project from the `Game` project allows flexibility in supporting multiple platforms, such as mobile.
* `MyNewProject.Game.Tests`: This project allows you to write and test the logic and UI of your project. It allows access to the Test Browser, which is a separate visual testing framework that will allow you to test your UI elements and other visuals.

## Game logic

If you open up `MyNewProject.Game/MyNewProjectGame.cs`, you'll see the class responsible for some basic game logic. It inherits from the `osu.Framework.Game` super-class and by default, implements a basic [screen stack](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks) setup:

```csharp
namespace MyNewProjectName.Game
{
    public partial class MainScreen : Screen
    {
        [BackgroundDependencyLoader]
        private void load()
        {
            InternalChildren = new Drawable[]
            {
                new Box
                {
                    Colour = Color4.Violet,
                    RelativeSizeAxes = Axes.Both,
                },
                new SpriteText
                {
                    Y = 20,
                    Text = "Main Screen",
                    Anchor = Anchor.TopCentre,
                    Origin = Anchor.TopCentre,
                    Font = FontUsage.Default.With(size: 40)
                },
                new SpinningBox
                {
                    Anchor = Anchor.Centre,
                }
            };
        }
    }
}
```

The main (and only) screen contains a spinning box visual:

```csharp
namespace MyNewProjectName.Game
{
    public partial class MainScreen : Screen
    {
        [BackgroundDependencyLoader]
        private void load()
        {
            InternalChildren = new Drawable[]
            {
                new Box
                {
                    Colour = Color4.Violet,
                    RelativeSizeAxes = Axes.Both,
                },
                new SpriteText
                {
                    Y = 20,
                    Text = "Main Screen",
                    Anchor = Anchor.TopCentre,
                    Origin = Anchor.TopCentre,
                    Font = FontUsage.Default.With(size: 40)
                },
                new SpinningBox
                {
                    Anchor = Anchor.Centre,
                }
            };
        }
    }
}
```

## Platform specific logic

The logic needed to begin execution of the game is located in `MyNewProject.Desktop/Program.cs`:

```csharp
namespace MyNewProjectName.Desktop
{
    public static class Program
    {
        public static void Main()
        {
            using (GameHost host = Host.GetSuitableDesktopHost(@"MyNewProjectName"))
            using (osu.Framework.Game game = new MyNewProjectNameGame())
                host.Run(game);
        }
    }
}
```

## Testing

### The test browser

`osu-framework` includes a visual testing framework that helps provide tests that can be verified both visually and systematically via [NUnit](https://nunit.org/). When running the tests project, a utility called the **test browser** will collect all of your tests, and present them in an executable that presents all of the tests in a list.
	
The pre-generated project will feature an example test class in `MyGameProject.Game.Tests/Visual/TestSceneMyNewProjectGame.cs`.
	
In order for your test browser to discover tests, you will need to specify a namespace in which for the browser to look in when constructing it:

```csharp
    public partial class MyNewProjectNameTestBrowser : MyNewProjectNameGameBase
    {
        protected override void LoadComplete()
        {
            base.LoadComplete();

            AddRange(new Drawable[]
            {
                new TestBrowser("MyNewProjectName"), // <- see namespace specification here
                new CursorContainer()
            });
        }

        // rest of class omitted for brevity
    }
}
```

Below is an example of a visual test that displays `MyNewProjectGame` in full inside the test browser:

```csharp
namespace MyNewProject.Game.Tests.Visual
{
    public partial class TestSceneMyNewProjectGame : TestScene
    {
        private MyNewProjectGame game;

        [BackgroundDependencyLoader]
        private void load(GameHost host)
        {
            game = new MyNewProjectGame();
            game.SetHost(host);

            AddGame(game);
        }
    }
}
```

Note that in order for you to be able to run the visual tests, you will have to switch your run configuration to the visual tests project. The solution template will generate the setup code necessary for this in `MyGameProject.Game.Tests/Program.cs`.

```csharp
namespace MyNewProjectName.Game.Tests
{
    public static class Program
    {
        public static void Main()
        {
            using (GameHost host = Host.GetSuitableDesktopHost("visual-tests"))
            using (var game = new MyNewProjectNameTestBrowser())
                host.Run(game);
        }
    }
}

```

## Adding tests to the Test Browser

Now that our Test Browser is discovering tests from the specified namespace, we can start adding tests! To do so, create a new class that derives [`TestScene`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Testing/TestScene.cs) with the [`TestFixture`](https://docs.nunit.org/articles/nunit/writing-tests/attributes/testfixture.html) attribute. From here, we can add steps to this test of various types. For information on what types of steps are available, please refer to [Dynamic Compilation and Visual Testing](https://github.com/ppy/osu-framework/wiki/Development-and-Testing#steps-and-automated-testing).

## Example Test

The following code adds a simple cube to the visual test browser that we created above. The cube has a rigid body attached, and should drop to the bottom of the screen when created. From here, we can choose to check the behavior of the cube by asserting that the cube eventually reaches the bottom via `AddAssert()`.

```csharp
namespace AwesomeGame.VisualTests
{
    [TestFixture]
    public class RigidCubeTest : TestScene
    {
        private RigidBodySimulation sim = null!;
        
        [BackgroundDependencyLoader]
        private void load()
        {
             // Set up the simulation once before any tests are ran
             Child = sim = new RigidBodySimulation { RelativeSizeAxes = Axes.Both };
        }

        [Test]
        public void AwesomeTestName()
        {
            AddStep("Drop a cube", performDropCube);
        }

        private void performDropCube()
        {
            // Add a new cube to the simulation
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

# Appendix

## `SetUp` attribute

The [`SetUp`](https://docs.nunit.org/articles/nunit/writing-tests/attributes/setup.html) NUnit attribute marks a method as a setup method that runs as a step before every group of tests in a test method. The steps created by this attribute get added to the visual test browser as well.

## `BackgroundDependencyLoader` attribute

The [`BackgroundDependencyLoader`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Allocation/BackgroundDependencyLoaderAttribute.cs) attribute denotes a method to be the load method of a [Drawable](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/Drawable.cs). You can specify a type in the method parameters to attempt to grab an object of that type that has been [cached](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Caching/Cached.cs).

## Further reading

- For information on how to load your own resources such as textures and audio, please read [Setting Up Compiled Resource Stores](https://github.com/ppy/osu-framework/wiki/Setting-Up-Compiled-Resource-Stores).
- For more information regarding dependency injection via the BackgroundDependencyLoader attribute, please read [Dependency Injection](https://github.com/ppy/osu-framework/wiki/Dependency-Injection)
- For additional reading on visual tests, please refer to [Dynamic Compilation and Visual Testing](https://github.com/ppy/osu-framework/wiki/Development-and-Testing)