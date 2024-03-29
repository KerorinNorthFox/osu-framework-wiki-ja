This tutorial assumes you have completed [setting up your first project](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project).

* [Adding your resources library to the Resource Store](https://github.com/ppy/osu-framework/wiki/Setting-Up-Compiled-Resource-Stores#adding-your-resources-library-to-the-resource-store)
* [Default resources structure](https://github.com/ppy/osu-framework/wiki/Setting-Up-Compiled-Resource-Stores#default-resources-structure-and-adding-new-resources)
  * [Adding custom fonts to the font store](https://github.com/ppy/osu-framework/wiki/Setting-Up-Compiled-Resource-Stores#adding-custom-fonts-to-the-font-store)
* [Accessing resources from the resource store](https://github.com/ppy/osu-framework/wiki/Setting-Up-Compiled-Resource-Stores#accessing-resources-from-the-resource-store)

# Adding your resources library to the Resource Store

The first time you set up your project, you will need to add a [`DllResourceStore`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/IO/Stores/DllResourceStore.cs) with the path to a resources library. This will enable you to load resources from the specified resource library at run-time, which then in turn allows caching to occur in the [`ResourceStore`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/IO/Stores/ResourceStore.cs). A few important resource stores are already created in the base [`Game`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Game.cs) class which all game instances should inherit.

By default, your resources will be compiled into a dll with your project name and copied into your output directory. This is convenient for our purposes, as all we need to do now is add a `DllResourceStore` to it using `Resources.AddStore()` in our `BackgroundDependencyLoader` method inside of our game class.

```csharp
[BackgroundDependencyLoader]
private void load()
{
     Resources.AddStore(new DllResourceStore(@"AwesomeGame.dll"));
}
```

# Default resources structure and adding new resources

[`Game`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Game.cs) will automatically create the following stores that point to the respective paths for each store:

* `/Textures` (`TextureStore`)
* `/Tracks` (`AudioManager.Tracks`)
* `/Samples` (`AudioManager.Samples`)
* `/Shaders` (`ShaderManager`)
* `/Fonts` (`FontStore`)

To add to these stores, simply add resources to the respective directories and specify them as `EmbeddedResources` inside your `.csproj`.

For example, to include your own Textures folder and all .png files inside of it recursively:

```xml
<ItemGroup>
 <EmbeddedResource Include="Textures\**\*.png" />
</ItemGroup>
```

## Adding custom fonts to the font store

osu!framework loads Open Sans into the [`FontStore`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/IO/Stores/FontStore.cs) by default. If you find that this is insufficient for your use, you may add your own fonts to the `FontStore` by adding your own [`GlyphStore`s](https://github.com/ppy/osu-framework/blob/master/osu.Framework/IO/Stores/GlyphStore.cs). This is preferably done at the initialization of your game within the `BackgroundDependencyLoader` method inside your game class by invoking the following:

```csharp
Fonts.AddStore(new GlyphStore(Resources, @"Fonts/AwesomeFont"));
```

This will cache our hypothetical "AwesomeFont" for use in any [`SpriteText`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/Sprites/SpriteText.cs), which can be set using the `Font` property.

# Accessing resources from the resource store

After having added your own resources, you can now access them via dependency injection. You will need to specify the type of store you're trying to inject in the method parameters. For example, if I have a texture named `awesomeTexture.png` inside of my `Textures` folder, I can load it like this:

```csharp
[BackgroundDependencyLoader]
private void load(TextureStore store)
{
   texture = store.Get(@"awesomeTexture");
}
```

osu!framework will then load the resources for the first time if they have not been cached yet.

To retrieve other resources from other resource stores, simply specify the type of resource store you wish to retrieve from the cache via the load() method's parameters. For example, to retrieve tracks from the audio store, the following code will be sufficient.

```csharp
ITrackStore tracks = null!;

[BackgroundDependencyLoader]
private void load(AudioManager audio)
{
   tracks = audio.Tracks;
}
```
