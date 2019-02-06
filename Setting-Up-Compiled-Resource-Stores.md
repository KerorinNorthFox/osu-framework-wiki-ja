# Setting Up Compiled Resource Stores

This tutorial assumes you have completed [setting up your first project](https://github.com/ppy/osu-framework/wiki/Setting-up-your-first-project).

The first time you set up your project, you will need to add a [DllResourceStore](https://github.com/ppy/osu-framework/blob/master/osu.Framework/IO/Stores/DllResourceStore.cs) with the path to a resources library. This will enable you to load resources from the specified resource library at run-time, which then in turn allows caching to occur in the [ResourceStore](https://github.com/ppy/osu-framework/blob/master/osu.Framework/IO/Stores/ResourceStore.cs). A few important resource stores are already created in the base [Game](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Game.cs) class which all Game instances should inherit.

## Adding your resources library to the Resource Store

By default, your resources will be compiled into a dll with your project name and copied into your output directory. This is convenient for our purposes, as all we need to do now is add a DllResourceStore to it using Resources.AddStore in our BackgroundDependencyLoader method inside of our game class.
```
[BackgroundDependencyLoader]
private void load()
{
     Resources.AddStore(new DllResourceStore(@"AwesomeGame.dll"));
}
```

## Default resources structure
[Game](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Game.cs) will automatically create the following stores that point to the respective paths for each store:

* /Textures (TextureStore)
* /Tracks 
* /Samples
* /Shaders
* /Fonts/OpenSans (FontStore)

To add to these stores, simply add resources to the respective directories and specify them as EmbeddedResources inside your .csproj.

For example, to include your own Textures folder and all .png files inside of it:
```
<ItemGroup>
 <EmbeddedResource Include="Textures\*.png" />
</ItemGroup>
```

## Accessing resources from the resource store
After having added your own resources, you can now access them via dependency injection. For example, if I have a texture named awesomeTexture.png inside of my Textures folder, I can load it like this:
```
[BackgroundDependencyLoader]
private void load(TextureStore store)
{
   texture = store.Get(@"awesomeTexture");
}
```

osu! framework will then load the resources for the first time if they have not been cached yet.