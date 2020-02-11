# Setting Up Fonts
This tutorial assumes you have completed [setting up your first project](/ppy/osu-framework/wiki/Setting-up-your-first-project) and [setting up compiled resource stores](Setting-Up-Compiled-Resource-Stores).

In this tutorial, you will learn how to add custom fonts into your game and use it:
* [Converting to binary font and texture files](#converting-to-bianry-font-and-texture-files)
* [Adding the custom font to the font store](#adding-the-custom-font-to-the-font-store)
* [Using specific font for sprite text](#using-specific-font-for-sprite-text)

## Converting to binary font and texture files
For the custom font to be recognized inside osu!framework, you will have to convert it into a binary font with texture files alongside it.

To start off, you'll need to download and install [BMFont by AngelCode](https://www.angelcode.com/products/bmfont) for converting the font, then run it and load [this configuration]() for selecting required character range:

| ![Load the required configurations #1](https://i.imgur.com/bnl9Eqk.png) | ![Load the required configurations #2](https://i.imgur.com/dhUJnK7.png)
|---|---|

---

Next, you have to set up the tool into using and converting your custom font properly by going into the font settings and selecting your custom font and check/uncheck `Bold` and `Italic`, and going into the export settings and selecting `Binary` as the font descriptor. **Without it, your game might hard crash when loading the font.**
| ![Set up tool settings #1](https://i.imgur.com/IcJNRGd.png) | ![Set up tool settings #2](https://i.imgur.com/XfD7Duq.png) |
|---|---|
| ![Set up tool settings #3](https://i.imgur.com/psnr8Nq.png) | ![Set up tool settings #4](https://i.imgur.com/Oz33p4Y.png) |

---

And lastly, converting the font can be accomplished by clicking on `Save bitmap font as` and saving it wherever you want inside your `Resources` folder, which then will be pointed out to later on:
| ![Save binary font #1](https://i.imgur.com/hznyByh.png) | ![Save binary font #2](https://i.imgur.com/FtsomGj.png) |
|---|---|

The font file name should be saved in the following pattern:
| `FontUsage.Family` | `FontUsage.Weight` | `FontUsage.Italics` | Font file name |
|--------------------|:------------------:|:-------------------:|:--------------:|
| MyAwesomeFont | `"Light"` | `false` | `MyAwesomeFont-Light` |
| MyAwesomeFont | `"Bold"` | `true` | `MyAwesomeFont-BoldItalic` |
| MyAwesomeFont | `null` | `false` | `MyAwesomeFont` |
| MyAwesomeFont | `null` | `true` | `MyAwesomeFont-Italic` |

## Adding the custom font to the font store
To use the custom font inside your game, you'll need to add it to the `FontStore` by specifying:
 - The resource store of your resources folder that contains the binary fonts.
 - The asset name of the binary font, The font path should be specified here relative from the resource store you've specified. (`Fonts/MyAwesomeFont` for this tutorial)

Here's an example of adding a custom font included within a `Resources` folder inside the game project: (`MyAwesomeProject.Game/Resources/Fonts/MyAwesomeFont`)
```csharp
namespace MyAwesomeProject.Game
{
    public class MyAwesomeGame : osu.Framework.Game
    {
        [BackgroundDependencyLoader]
        private void load()
        {
            // Add the DLL resource store of this game project into the global resources store.
            Resources.AddStore(new DllResourceStore(@"MyAwesomeProject.dll"));

            // Add the custom font to the font store.
            AddFont(Resources, @"Resources/Fonts/MyAwesomeFont");
        }
    }
}
```

---

Now you're almost done, by default, the first font to be added to your game will automatically be used for all `SpriteText`s in your game, but you may want to use a specific font.
## Using specific font for a sprite text
To use a specific font for a sprite text, all you have to do is construct a new `FontUsage` with:
| Parameter | Description | Value in this tutorial |
| --------- | ----------- | ---------------------- |
| `family` | The font name | `MyAwesomeFont` |
| `weight` | The weight of the font in string | `null` |
| `italics` | Whether the font is of italic type | `false` |

Check the font file name table example above for learning more.

There are also other `FontUsage` properties you can modify for rendering the font differently (e.g. `size` and `fixedWidth`)

This opens up for a lot of ways to implement the usage of your custom font, either by adding static properties for constructing the `FontUsage` or adding a static function with enumeration parameters that you can pick your font and its properties from, all based on your preference.

Here's an example of creating a sprite text using the custom font with `size: 40f`:
| ![Code](https://i.imgur.com/1MV0AZC.png) | ![Preview](https://i.imgur.com/LnohnKj.png)
|---|---|