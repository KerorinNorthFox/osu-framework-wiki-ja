Occasionally we will make changes which require consumers of the framework to make amendments to maintain compatibility. Wherever possible, we maintain compatibility via `[Obsolete]` attributes, but it is encouraged that you leave obsolete warnings turned on, and deal with them sooner rather than later. Generally we aim to leave obsolete methods around for 3-6 months after obsoletion.

This page serves to give a list of all breaking/major changes.

# vNext

## `Storage.Move()` will overwrite file at target destination if present

Previously it would fail if a file already existed at the target destination. Overwriting is usually the preferred outcome.

# [2023.822.0](https://github.com/ppy/osu-framework/releases/tag/2023.822.0)

## Maskable vertex input layout has changed

Vertices which want to use the built-in `VertexShaderDescriptor.TEXTURE` fragment shader, or the `sh_Masking.h` particle, will need to adjust the vertex input slightly:

```diff
+ #include "Internal/sh_MaskingInfo.h"

layout(location = 0) in vec2 m_Position;
+ layout(location = 1) in int m_MaskingIndex;

+ layout(location = 5) flat out int v_MaskingIndex;
+ layout(location = 6) out highp vec2 v_ScissorPosition;

void main(void)
{
+   InitMasking(m_MaskingIndex);

    // Transform from screen space to masking space.
-   highp vec3 maskingPos = g_ToMaskingSpace * vec3(m_Position, 1.0);
+   highp vec4 maskingPos = g_MaskingInfo.ToMaskingSpace * vec4(m_Position, 1.0, 0.0);
    v_MaskingPosition = maskingPos.xy / maskingPos.z;

+   // Transform from screen space to scissor space.
+   highp vec4 scissorPos = g_MaskingInfo.ToScissorSpace * vec4(m_Position, 1.0, 0.0);
+   v_ScissorPosition = scissorPos.xy / scissorPos.z;

+   v_MaskingIndex = m_MaskingIndex;

    gl_Position = g_ProjMatrix * vec4(m_Position, 1.0, 1.0);
}
```

Likewise, the C# definition must also be updated to write the `m_MaskingIndex` input, and be constructed with an `IRenderer`:

```diff
[StructLayout(LayoutKind.Sequential)]
public struct MyCustomVertex : IEquatable<MyCustomVertex>, IVertex
{
    [VertexMember(2, VertexAttribPointerType.Float)]
    public Vector2 Position;

+   [VertexMember(1, VertexAttribPointerType.Int)]
+   private readonly int maskingIndex;

+   public MyCustomVertex(IRenderer renderer)
+   {
+       this = default;
+       maskingIndex = renderer.CurrentMaskingIndex;
+   }

    public readonly bool Equals(MyCustomVertex other) =>
        Position.Equals(other.Position)
+       && maskingIndex == other.maskingIndex;
}
```

## Non-masking fragment shaders must use non-masking vertex shaders

Any usages of `VertexShaderDescriptor.TEXTURE_2` as a vertex shader in-combination with a non-masking fragment shader (i.e. one that does not use the built-in `sh_Masking.h` particle), should be updated to use `VertexShaderDescriptor.TEXTURE_2_NO_MASKING` instead.

## `IRenderer.PushScissorOffset()`/`IRenderer.PopScissorOffset()` have been removed

The way masking applies scissor has changed such that this no longer has any use.

## `MaskingInfo` layout has changed

- `ScreenSpaceAABB` renamed to `ScreenSpaceScissorArea`.
- `MaskingRect` renamed to `MaskingArea`
- `ToScissorSpace` added. If `ScreenSpaceScissorArea` truly represents a screen-space area, set this to `Matrix3.Identity`, otherwise set it to convert vertex coordinate inputs to the appropriate coordinate space of `ScreenSpaceScissorArea`.

# [2023.817.0](https://github.com/ppy/osu-framework/releases/tag/2023.817.0)

## `TexturedVertex2D` must be initialised with an `IRenderer`.

The default constructor for this type has been obsoleted and will now generate a compiler error.

# [2023.720.0](https://github.com/ppy/osu-framework/releases/tag/2023.720.0)

## Source generators will now only run on release builds

As we continue to add more source generators, we've seen increases in compile-time overheads, with local testing showing over 2x compile times with source generators turned on.

osu!framework source generators are made to optimise builds at runtime (mostly by removing reflection overhead). As such, it doesn't make sense to run these for debug releases as they are basically custom release-targeting optimisations.

This should not result in a noticeable change in runtime performance during debug, but will reduce compilation times by over 50% in most cases. This is valuable during debug as the most common case is frequently building / hot reloading for quick iteration.

# [2023.714.0](https://github.com/ppy/osu-framework/releases/tag/2023.714.0)

## `GameHost.GetClipboard()` is obsolete

The new way to retrieve a `Clipboard` instance is to resolve it via dependency injection directly. This does not require resolving the `GameHost` anymore:

```diff
 [BackgroundDependencyLoader]
-private void load(GameHost host)
+private void load(Clipboard clipboard)
 {
-    host?.Clipboard.SetText("example");
+    clipboard.SetText("example");
 }
```

# [2023.618.0](https://github.com/ppy/osu-framework/releases/tag/2023.618.0)

## `AddDropdownItem(LocalisableString text, T value)` has been removed

If you were using this, refactor your code to use `LocalisableString GenerateItemText(T value)` instead.

Easiest way is to add `LocalisableString? CustomDropdownText` to your `T`, or wrap `T` in a class that has one.
Then simply change `GenerateItemText()` to use that:

```cs
public partial class MyDropdown : Dropdown<T>
{
    protected override LocalisableString GenerateItemText(T value)
        => value.CustomDropdownText ?? base.GenerateItemText(value);
}
```

## Subclasses of `FocusedOverlayContainer` must now implement `Pop{In,Out}()`

Previously, subclasses of `FocusedOverlayContainer` were not forced to implement `Pop{In,Out}` themselves, despite actually needing to do so for the overlay to actually play transitions out correctly. This has now been changed and inheritors of the class must always implement `Pop{In,Out}`.

In practice this should not break any reasonable consumers of the class, except for the need to remove any `base.Pop{In,Out}()` calls in classes that directly inherit `FocusedOverlayContainer` and call base in `Pop{In,Out}()`.

# [2023.326.0](https://github.com/ppy/osu-framework/releases/tag/2023.326.0)

## iOS project structure has been revamped

In efforts to migrate the framework to SDL on iOS, the project structure required major changes. For consumers, you're required to remove the `AppDelegate` class attached in the game project as `GameAppDelegate` no longer exists, and change the `Application.Main` method to call `GameApplication.Main` as such:
```diff
-        public static void Main(string[] args) => UIApplication.Main(args, null, typeof(AppDelegate));
+        public static void Main(string[] args) => GameApplication.Main(new MyGameIOS());
```

# [2023.322.0](https://github.com/ppy/osu-framework/releases/tag/2023.322.0)

## `TexturedShaderDrawNode.TextureShader` is made private

This is partially done to force consumers into binding uniform blocks in the newly defined `TexturedShaderDrawNode.BindUniformResources` method, such that uniform blocks are always bound whenever the corresponding shader is used for drawing in the framework.

For binding/unbinding shaders, use `TexturedShaderDrawNode.BindTextureShader`/`TexturedShaderDrawNode.UnbindTextureShader` instead.

# [2023.314.0](https://github.com/ppy/osu-framework/releases/tag/2023.314.0)

## The OpenGL Core profile is now used

Shaders now follow "Vulkan GLSL", which is mostly documented in the spec: [`GL_KHR_vulkan_glsl`](https://github.com/KhronosGroup/GLSL/blob/master/extensions/khr/GL_KHR_vulkan_glsl.txt). The primary differences are documented below:

### Vertex shader members

<table>
<tr>
<th>old</th>
<th>new</th>
<tr>
<td>

```glsl
attribute lowp int In_Value;
varying lowp int Out_Value;
```

</td>
<td>

```glsl
layout(location = 0) in lowp int InValue;
layout(location = 0) out lowp int OutValue;
```

</td>
</tr>
</table>

The location increases for each additional member in the respective `in`/`out` lists.

### Fragment shader members

<table>
<tr>
<th>old</th>
<th>new</th>
<tr>
<td>

```glsl
varying lowp int In_Value;
```

</td>
<td>

```glsl
layout(location = 0) in lowp int InValue;
```

</td>
</tr>
</table>

The location increases for each additional member, and must match the location of the respective member in the vertex shader.

### Fragment shader outputs

Fragment shaders are required to define an output variable.

<table>
<tr>
<th>old</th>
<th>new</th>
<tr>
<td>

```glsl
void main(void)
{
    gl_FragColor = vec4(0.0);
}
```

</td>
<td>

```glsl
layout(location = 0) out vec4 colour;

void main(void)
{
    colour = vec4(0.0);
}
```

</td>
</tr>
</table>

### Uniform members

Free-floating "global" uniforms are not supported. They must be placed in ["uniform blocks"](https://www.khronos.org/opengl/wiki/Interface_Block_(GLSL)#Buffer_backed).

<table>
<tr>
<th>old</th>
<th>new</th>
<tr>
<td>

```glsl
uniform lowp int Uniform_Value;
```

</td>
<td>

```glsl
layout(std140, set = 0, binding = 0) uniform MyUniforms
{
    lowp int Uniform_Value;
};
```

</td>
</tr>
</table>

**Notes:**
- `std140` should always be used.
- The `set` increases for each unique uniform block.
- `binding` is a special required value which indicates where in the program the block should be bound. osu!framework manages this for you and should be 0 for uniform blocks.
- Both shader stages can use the same uniform block from the same set, provided that the physical layout matches.

Furthermore, this means that uniform blocks have become a first-class citizen in osu!framework. The code to make use of the above looks like this:
```csharp

class MyDrawNode : DrawNode
{
    private IUniformBuffer<MyBufferData>? bufferData;

    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

        // Create the uniform buffer.
        bufferData ??= renderer.CreateUniformBuffer<MyBufferData>();

        // Update the uniform buffer.
        bufferData.Data = bufferData.Data with
        {
            UniformValue = 5
        };

        // Bind the buffer to the shader.
        shader.BindUniformBuffer("MyUniforms", bufferData);

        shader.Bind();
        // Draw...
    }

    protected override void Dispose(bool isDisposing)
    {
        base.Dispose(isDisposing);

        // Ensure that the buffer is disposed.
        bufferData?.Dispose();
    }
}

// Pack = 1 is required.
[StructLayout(LayoutKind.Sequential, Pack = 1)]
// Record structs automatically implement IEquatable<>
public record struct MyBufferData
{
    // Use types provided in the osu.Framework.Graphics.Shaders.Types namespace.
    public UniformInt UniformValue;

    // The struct must be a multiple of 16 bytes.
    private readonly UniformPadding12 pad2;
}
```

Runtime validation is present to notify of incorrect alignment/packing.

### Uniform textures/samplers

The combined sampler of GLSL is not supported.

<table>
<tr>
<th>old</th>
<th>new</th>
<tr>
<td>

```glsl
uniform lowp sampler2D Sampler;
```

</td>
<td>

```glsl
layout(set = 0, binding = 0) uniform lowp texture2D Texture;
layout(set = 0, binding = 1) uniform lowp sampler Sampler;
```

</td>
</tr>
</table>

**Notes:**
- `std140` cannot be applied here unlike for uniform blocks.
- The `set` increases along with all other uniform blocks/textures.
- The texture and sampler are bound to different points. The texture being bound to 0 and sampler to 1 will be standard for osu!framework going forward.

### Sampling textures

<table>
<tr>
<th>old</th>
<th>new</th>
<tr>
<td>

```glsl
vec4 col = texture2D(my_Sampler, ...);
```

</td>
<td>

```glsl
vec4 col = texture(sampler2D(my_Texture, my_Sampler), ...);
```

</td>
</tr>
</table>

# [2022.1129.0](https://github.com/ppy/osu-framework/releases/tag/2022.1129.0)

## `IHasFilterableChildren` has been removed; use `IFilterable` instead (https://github.com/ppy/osu-framework/pull/5530)

Rather than rely on `IHasFilterableChildren` to descend down complex hierarchies for filtering purposes, `IContainerEnumerable<T>.Children` is used, which results in less boilerplate as you don't need to remember to implement it whenever needed.

## `IConditionalFilterable` is introduced (https://github.com/ppy/osu-framework/pull/5530)

By implementing `IConditionalFilterable`, you get one extra bindable to play with. If the value of said bindable is `true`, then the item is included in textual search as usual. If it is `false`, however, it - and its entire subtree - is excluded from further search on the premise that the item does not meet external criteria. In practical terms, you would set it to `false` when you want to hide some items inside a `SearchContainer` because they're unavailable due to other settings.

## Classes used in dependency injection need to implement the `IDependencyInjectionCandidate` interface (https://github.com/ppy/osu-framework/pull/5548)

This does not affect `Drawable` subclasses.

# [2022.1126.0](https://github.com/ppy/osu-framework/releases/tag/2022.1126.0)

## Add source generator for dependency injection (https://github.com/ppy/osu-framework/pull/5541)

This one's been brewing for a long time while I tried to figure out the best way to do things. It provides a structural foundation on top of which we can build other source generators.

For source generators to work at all for us, we unfortunately need to make every `Drawable` class `partial`. This can be done as a graceful upgrade – previous reflection pathways are still supported so no immediate changes should be required to your project to compile.

### Analyser + codefix

I've created a pretty "basic" analyser for warning against non-`partial` classes when required. It attempts to discover:
- Types used as the argument to calls of `DependencyContainer.Inject()`.
- Types used as type arguments of `CachedModelDependencyContainer<T>`.
- Types implementing `ITransformable`, `IDrawable` or `ISourceGeneratedDependencyActivator`.

This covers all current cases while being as least invasive as possible (i.e. not analysing every member of every class ever).

A code fix has been provided for the diagnostic, however if you're planning to codefix the whole solution I suggest the running the following command until it outputs 0 changes:

```
dotnet format analyzers osu.Desktop.slnf --verbosity d
```

The code fix seems to not fix all issues on the first attempt. This is potentially caused by my implementation, however I've tried everything that I could find and nothing's worked. I'll look into fixing this as a future effort, however for the most part it's a one-time thing.

The analyser and code fixer don't have tests at the moment. These are also planned in a future effort and will be structured similarly to the source generator tests and the `osu-localisation-analyser` project.

### Notes

- Member accessor validation is not supported yet. I think these will probably be added to an analyser in a future effort, but I've disabled the relevant tests in https://github.com/smoogipoo/osu-framework/commit/f3324205ded5ed1c05fa3a92a8fc4b5c26ec7cb4 for now.
- Build time has increased. This can probably be improved further, however I don't want to drag on this PR much longer.
- Going to definitions of classes is a +1 process now, with every single definition now giving you two locations.
- I eventually want to update the source generator to make use of `IIncrementalGenerator`, but there's a bit more reading to do for that.
- The class is generated via AST rather than raw code. This is a personal/stylistic choice because I originally implemented it using raw code and couldn't bear it.
- The SG code quality is not quite where I want it to be at the moment - it's lacking documentation (compared to something like `osu-localisation-analyser`) and the tests are lacking structure. This will be improved in a future effort.

I have not tested VSCode compatibility, but I have no reason to believe it won't work, unlike `osu-localisation-analyser` where there's differences in how IDEs add files to workspaces from analysers/codefixes.

### Performance

Check the pull request thread for CPU and memory profiling, but the short of this is that framework allocations required for dependency injection are down 50% or more; overall allocations at runtime are down 50% (by object count) and also substantially by memory; CPU overhead for DI activation (ie. creating a new `Drawable`) is down magnitudes on first usage, and also marginally on subsequent usages.

Put simply, it's a win all-round.

## `Bindable<T>.CopyTo` should be implemented for any custom bindables (https://github.com/ppy/osu-framework/pull/5531)

To improve the performance of creating clones of bindables (ie. via `GetUnboundCopy()`), the copy portion of `BindTo()` has been split out into its own method. If you have any custom bindable types which were copying values across within a `BindTo()` override, please move the copy operation to `CopyTo()` instead. Pay special attention to the direction of assignment, which will reverse from what it was in `BindTo`.

A fix should look something like this:

```diff
diff --git a/osu.Framework/Bindables/BindableNumber.cs b/osu.Framework/Bindables/BindableNumber.cs
index 6d605667f..430da79d0 100644
--- a/osu.Framework/Bindables/BindableNumber.cs
+++ b/osu.Framework/Bindables/BindableNumber.cs
@@ -199,12 +199,12 @@ protected void TriggerPrecisionChange(BindableNumber<T> source = null, bool prop
                 PrecisionChanged?.Invoke(precision);
         }
 
-        public override void BindTo(Bindable<T> them)
+        public override void CopyTo(Bindable<T> them)
         {
             if (them is BindableNumber<T> other)
-                Precision = other.Precision;
+                other.Precision = Precision;
 
-            base.BindTo(them);
+            base.CopyTo(them);
         }
 
         public override void UnbindEvents()

```

# [2022.907.0](https://github.com/ppy/osu-framework/releases/tag/2022.907.0)

## `DepthStencilFunction` renamed to `BufferTestFunction`

The new naming should better fit future usages outside of depth and stencil tests.

# [2022.901.0](https://github.com/ppy/osu-framework/releases/tag/2022.901.0)

## `CompositeDrawable.RemoveInternal`, `IContainer.Remove`, `IContainer.RemoveRange` and `IContainer.RemoveAll` now require a `bool` parameter

A common mistake we've seen made osu!-side is where drawables are `Remove`d from the hierarchy with the intention of never using them again. In such cases, disposal is not guaranteed – nor is unbinding of `Bindable` fields/properties – which can lead to event and object leakage.

To prevent this from happening, a new `disposeImmediately` parameter has been added.

Generally this should be set to `true` unless you intend to reuse the removed drawable (ie. by adding back to the hierarchy at a different location).

# [2022.816.0](https://github.com/ppy/osu-framework/releases/tag/2022.816.0)

## Depth test function is now typed to `DepthStencilFunction`

In places where the `DepthInfo` struct is used with a custom depth function, the following changes are required.
```diff
- new DepthInfo(function: DepthFunction.Never)
+ new DepthInfo(function: DepthStencilFunction.Never)

- new DepthInfo(function: DepthFunction.Less)
+ new DepthInfo(function: DepthStencilFunction.LessThan)

- new DepthInfo(function: DepthFunction.Lequal)
+ new DepthInfo(function: DepthStencilFunction.LessThanOrEqual)

- new DepthInfo(function: DepthFunction.Equal)
+ new DepthInfo(function: DepthStencilFunction.Equal)

- new DepthInfo(function: DepthFunction.Gequal)
+ new DepthInfo(function: DepthStencilFunction.GreaterThanOrEqual)

- new DepthInfo(function: DepthFunction.Greater)
+ new DepthInfo(function: DepthStencilFunction.GreaterThan)

- new DepthInfo(function: DepthFunction.Notequal)
+ new DepthInfo(function: DepthStencilFunction.NotEqual)

- new DepthInfo(function: DepthFunction.Always)
+ new DepthInfo(function: DepthStencilFunction.Always)
```

# [2022.805.0](https://github.com/ppy/osu-framework/releases/tag/2022.805.0)

## `IRenderer` added as parameter to `DrawNode`

In order to allow the framework to interoperate with different rendering backends, all rendering functions are now performed through a new `IRenderer` interface and parameter.

```diff
- DrawNode.Draw(Action<TexturedVertex2D> vertexAction);
+ DrawNode.Draw(IRenderer renderer);
- DrawNode.DrawOpaqueInterior(Action<TexturedVertex2D> vertexAction);
+ DrawNode.DrawOpaqueInterior(IRenderer renderer);
```

In places where the `vertexAction` parameter was previously used (e.g. to draw textures), `null` can be given as a parameter instead.

## `GLWrapper` removed, replaced by calls through `IRenderer`

The static `GLWrapper` class has been removed, and all drawing functions should be performed via the new `DrawNode` parameter instead. There is a 1-1 API correlation between the classes.

```diff
public class MyDrawNode : DrawNode
{
    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

-       GLWrapper.PushDepthInfo(...);
+       renderer.PushDepthInfo(...);

-       GLWrapper.SetBlend(...);
+       renderer.SetBlend(...);

-       GLWrapper.PopDepthInfo();
+       renderer.PopDepthInfo();
    }
}
```

## `Texture.WhitePixel` has moved to `IRenderer`

For cases where it's used as a fallback texture, it can be retrieved by [resolving](/ppy/osu-framework/wiki/Dependency-Injection) an `IRenderer` into the `Drawable` class and accessing `IRenderer.WhitePixel`.

```diff
public class MyDrawable : Drawable
{
+   [BackgroundDependencyLoader]
+   private void load(IRenderer renderer)
+   {
+       texture ??= renderer.WhitePixel;
+   }

    private Texture texture;

    public Texture Texture
    {
-       get => texture ?? Texture.WhitePixel;
+       get => texture;
        set => texture = value;
    }
}
```

## Texture drawing methods moved from `DrawNode` to `IRenderer` extension methods

Appearing alongside `IRenderer` is the new `RendererExtensions` class providing helper methods for common drawing procedures.
```diff
public class MyDrawNode : DrawNode
{
    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

-       DrawQuad(renderer.WhitePixel, ...);
+       renderer.DrawQuad(renderer.WhitePixel, ...);
    }
}
```
All of the following methods have been moved to this class:
```
DrawTriangle()
DrawQuad()
DrawClipped()
DrawFrameBuffer()
```

## Textures must be created through the `IRenderer`

[Resolve](/ppy/osu-framework/wiki/Dependency-Injection) an `IRenderer` into the `Drawable` class and use `IRenderer.CreateTexture()` method to create textures.

```diff
public class MyDrawable : Drawable
{
-   private readonly Texture texture;

-   public MyDrawable()
-   {
-       texture = new Texture(100, 100);
-       texture.SetData(...);
-   }
+   private Texture texture;

+   [BackgroundDependencyLoader]
+   private void load(IRenderer renderer)
+   {
+       texture = renderer.CreateTexture(100, 100);
+       texture.SetData(...);
+   }
}
```

`DummyRenderer` may be used for cases where textures need to be created and neither an `IRenderer` nor `GameHost` is accessible:

```diff
[TestFixture]
public class MyTestFixture
{
    public void CreateATexture()
    {
-       Texture texture = new Texture(1, 1);
+       Texture texture = new DummyRenderer().CreateTexture(1, 1);
    }
}
```

## `TextureGL` is no longer accessible

Properties such as `TextureGL.BypassTextureUploadQueueing` have been moved to `Texture` itself, and `Texture` can be used for all rendering procedures. When rendering, textures are now bound to different sampling units via an integer value.

```diff
public class MyDrawable : Drawable
{
    private Texture texture;

    [BackgroundDependencyLoader]
    private void load(IRenderer renderer)
    {
        texture = renderer.CreateTexture(100, 100);
-       texture.TextureGL.BypassUploadQueueing = true;
+       texture.BypassUploadQueueing = true;
        texture.SetData(...);
    }
}

public class MyDrawNode : DrawNode
{
    private Texture texture;

    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

-       texture.TextureGL.Bind();
-       texture.TextureGL.Bind(TextureUnit.Texture1);
+       texture.Bind();
+       texture.Bind(1);

        // Note: The texture binding above isn't required in either case for the call below.
-       DrawQuad(texture.TextureGL, ...);
+       renderer.DrawQuad(texture, ...);
    }
}
```

## The `QuadBatch<T>` and `LinearBatch<T>` classes are no longer accessible

Create vertex batches via the `IRenderer` and store as an `IVertexBatch<T>` instead:

```diff
public class MyDrawNode : DrawNode
{
-   private readonly QuadBatch<TexturedVertex2D> quadBatch = new QuadBatch<TexturedVertex2D>(1, 1);
-   private readonly LinearBatch<TexturedVertex2D> linearBatch = new LinearBatch<TexturedVertex2D>(1, 1, PrimitiveType.Triangles);
+   private IVertexBatch<TexturedVertex2D> quadBatch;
+   private IVertexBatch<TexturedVertex2D> linearBatch;

    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

+       quadBatch ??= renderer.CreateQuadBatch<TexturedVertex2D>(1, 1);
+       linearBatch ??= renderer.CreateLinearBatch<TexturedVertex2D>(1, 1, PrimitiveTopology.Triangles);
    }

    protected override void Dispose(bool disposing)
    {
        base.Dispose(disposing);
        batch?.Dispose();
    }
}
```

Beware that when creating linear batches, the type parameter has changed from `PrimitiveType` to `PrimitiveTopology`!

## The `FrameBuffer` class is no longer accessible

Create frame buffers via the `IRenderer` and store as an `IFrameBuffer`.

```diff
public class MyDrawNode : DrawNode
{
-   private readonly FrameBuffer myFrameBuffer = new FrameBuffer(new[] { RenderbufferInternalFormat.DepthComponent16 }, All.Nearest);
+   private IFrameBuffer myFrameBuffer;

    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

+       myFrameBuffer ??= renderer.CreateFrameBuffer(new[] { RenderBufferFormat.D16 }, TextureFilteringMode.Nearest);
    }

    protected override void Dispose(bool disposing)
    {
        base.Dispose(disposing);
        myFrameBuffer?.Dispose();
    }
}
```

Beware that the render buffer type parameter has changed from `RenderbufferInternalFormat` to `RenderBufferFormat`!

## The `Shader` class is no longer accessible

Shaders are still created via the `ShaderManager`, but are now returned as `IShader`s.

```diff
public class MyDrawable : Drawable
{
-   private Shader shader;
+   private IShader shader;

    [BackgroundDependencyLoader]
    private void load(ShaderManager shaders)
    {
        shader = shaders.Load("a", "b");
    }
}
```

## `TextureStore`, `FontStore` and `LargeTextureStore` require an `IRenderer` constructor parameter

One common use case is to provide a new texture store from inside a derived `Game`, for which the following change is required:

```diff
public class TestGame : osu.Framework.Game
{
    private DependencyContainer dependencies;

    protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent) =>
        dependencies = new DependencyContainer(base.CreateChildDependencies(parent));

    [BackgroundDependencyLoader]
    private void load()
    {
-       var largeStore = new LargeTextureStore(Host.CreateTextureLoaderStore(new NamespacedResourceStore<byte[]>(Resources, @"Textures")), All.Nearest);
+       var largeStore = new LargeTextureStore(Host.Renderer, Host.CreateTextureLoaderStore(new NamespacedResourceStore<byte[]>(Resources, @"Textures")), TextureFilteringMode.Nearest);
        largeStore.AddTextureSource(Host.CreateTextureLoaderStore(new OnlineStore()));
        dependencies.Cache(largeStore);
    }
}
```

Beware that the filter mode parameter has changed from `All` to `TextureFilteringMode`!

## Some `TexturedShaderDrawNode` members now take an `IRenderer` parameter

```diff
public class MyDrawNode : TexturedShaderDrawNode
{
    public override void Draw(IRenderer renderer)
    {
        base.Draw(renderer);

-       Shader.Bind();
+       var shader = GetAppropriateShader(renderer);
+       shader.Bind();

        // ...

-       Shader.Unbind();
+       shader.Unbind();
    }

-   protected override bool RequiresRoundedShader => ...;
+   protected override bool RequiresRoundedShader(IRenderer renderer) => ...;
}

```

## `IVertex`, `TexturedVertex2D`, et al. have been re-namespaced

They are now placed under the `osu.Framework.Graphics.Rendering.Vertices` namespace

## Several enums have been re-namespaced

```
WrapMode    -> osu.Framework.Graphics.Textures.WrapMode
Opacity     -> osu.Framework.Graphics.Textures.Opacity
ClearInfo   -> osu.Framework.Graphics.Rendering.ClearInfo
MaskingInfo -> osu.Framework.Graphics.Rendering.MaskingInfo
DepthInfo   -> osu.Framework.Graphics.Rendering.DepthInfo
```

# [2022.624.0](https://github.com/ppy/osu-framework/releases/tag/2022.624.0)

## `TextureStore.AddStore/RemoveStore` have been split into "store" and "texture source" methods

In an effort to make the texture API easier to comprehend, the `AddStore`/`RemoveStore` methods have been split into two pairs:
 - `AddTextureSource`/`RemoveTextureSource`, for adding/removing texture data lookup sources (i.e. `TextureLoaderStore`s)
 - `AddStore`/`RemoveStore`, for adding/removing `TextureStore`s.

Any existing usages of `AddStore` for adding lookup sources must be changed to use `AddTextureSource` instead.

# [2022.607.0](https://github.com/ppy/osu-framework/releases/tag/2022.607.0)

## "Unlimited" frame limiter is no longer completely unlimited #5235

Games using osu!framework can generally run at *very* high frame rates when not much is going on.

This can be counter-productive due to the induced allocation and GPU overhead.
 - Allocation overhead can lead to excess garbage collection
 - GPU overhead can lead to unexpected pipeline blocking (and stutters as a result).
   Also, in general graphics card manufacturers do not test their hardware at insane frame rates and therefore drivers are not optimised to handle this kind of throughput.
 - We only harvest input at 1000hz, so running any higher has zero benefits.

If you think you know better for your specific application (or more correctly need to remove the limit for benchmarking), set `GameHost.AllowBenchmarkUnlimitedFrames` to `true`.

# [2022.528.0](https://github.com/ppy/osu-framework/releases/tag/2022.528.0)

## `IHasFilterTerms.FilterTerms` is now an array of `LocalisableString`s

Supports filtering by either the original text or the localised form according to the currently selected language. Migration:
```diff
- public IEnumerable<string> FilterTerms => new string[] { ... };
+ public IEnumerable<LocalisableString> FilterTerms => new LocalisableString[] { ... };
```

## `ScrollEvent.ScrollDelta.X` (horizontal scrolling) is now consistent across platforms.

Previously, Windows, Linux and Android were inverted. Delta is positive when mouse wheel scrolled to the up or left, in non-"natural" scroll mode (ie. the classic way).

# [2022.421.0](https://github.com/ppy/osu-framework/releases/tag/2022.421.0)

## `IScreen` navigation interface methods now receive an "event args" structure

To allow adding more data to `IScreen` navigation interface methods in the future without further API breakage, their signatures have been changed to include an "event args" structure in the following manner:

```diff
- void OnEntering(IScreen last);
+ void OnEntering(ScreenTransitionEvent e);

- void OnExiting(IScreen next);
+ void OnExiting(ScreenExitEvent e);

- void OnResuming(IScreen last);
+ void OnResuming(ScreenTransitionEvent e);

- void OnSuspending(IScreen next);
+ void OnSuspending(ScreenTransitionEvent e);
```

The `last` and `next` arguments from the old signatures can now be accessed via `e.Last` and `e.Next` respectively.

Additionally, `ScreenExitEvent` contains a new `Destination` member, that allows to specify which screen is the "destination" screen of an exit operation spanning multiple screens.

# [2022.223.0](https://github.com/ppy/osu-framework/releases/tag/2022.223.0)

## Visual test projects now use native .NET 6 "Hot reload"

Over the years we have maintained our own version of hot reload, affectionately named "dynamic compilation". Even after multiple complete rewrites of the system, there are still edge cases where it will unexpectedly fall over due to being too greedy in including what it considers required for the recompile.


The startup cost when running in debug was considerable (around 10-30s initialisation) and the first-compile overhead could also be high (5-60s). In addition, the dependencies required to make it work increased the final assembly size of osu!framework, even for release deploys.

With the introduction of cross-platform support for hot reload in .NET 6 we have made the decision to switch to the natively supported version.

To use this new version:
- From Rider, you'll get a popup that you have to click "Apply Changes" when a change is made to code while running. You can bind a key to `Apply Hot Reload Changes` in Rider (defaults to Alt+F10) to make it quicker to apply changes.
- Running `dotnet watch` from CLI on your test project will automatically watch files and recompile when a hange is seen.

New limitations:
- The limitations of hot reload are [listed here](https://github.com/dotnet/roslyn/blob/main/docs/wiki/EnC-Supported-Edits.md).
- Notably, adding new `override` methods or `class`es are not supported. You can workaround this for the common scenario of prototyping new components by creating subclasses and adding the `override` methods before the initial hot reload.

We are interested in hearing feedback on this change, especially troubling cases where the previous behaviour worked better for you. Hope is that the limitations of the new method are outweighed by the leaner assembly, better performance, and (in general) up-front error when a change can't be applied, rather than an error that can be delayed longer than it would take to run a full recompile/restart.

# [2022.214.0](https://github.com/ppy/osu-framework/releases/tag/2022.214.0)

## `InputManager.ChangeFocus()` will no longer switch focus to drawables that are not alive, not present or do not have a parent

To avoid unusual scenarios concerning `ChangeFocus()`, wherein a drawable could potentially request focus and have focus automatically taken away from it every frame, `ChangeFocus()` now checks whether the target drawable is alive, present and has a parent before switching focus to it.

This potentially breaks scenarios such as calling `ChangeFocus()` in `LoadComplete()` on a child drawable with the expectation that the child drawable should receive focus as soon as its ancestor is added to the draw hierarchy. In such scenarios, the suggested fix is to schedule the `ChangeFocus()` operation after children so that it is performed only when the child is fully prepared to receive focus.

In general it is recommended to check the return value of `ChangeFocus()` to determine as to whether focus was actually changed.

## `CompositeDrawable.BorderColour` has changed type from `SRGBColour` to `ColourInfo`

To facilitate gradiented border support, the type of `CompositeDrawable.BorderColour` has changed from `SRGBColour` to `ColourInfo`. While some implicit conversions from `SRGBColour` to `ColourInfo` exist, some properties of `SRGBColour` are not available on `ColourInfo`, as the latter does not always represent a single colour, and may require appropriate adjustments.

# [2021.1118.0](https://github.com/ppy/osu-framework/releases/tag/2021.1118.0)

## `KeyBindingContainer` now sends key repeat events by default

`KeyBindingContainers` are commonly used in UI, where we expect to be able to receive key repeats. Historically this was controlled by the `SendRepeats` flag, but now that all events send `KeyBindingPressEvent` with a repeat argument, handling this legacy mode of operation was causing more issues that it was worth. The default expectation should be that repeats arrive.

For cases where key repeat may not be wanted (ie. gameplay, where your code is in complete control of user input and doesn't want to receive arbitrary key repeat events), you may opt out of receiving them by creating a subclass of `KeyBindingContainer`:

```csharp
public class NoRepeatKeyBindingContainer : KeyBindingContainer<T>
{
    protected override bool HandleRepeats => false;
}
```

Alternatively, if you have only certain cases which you wish to opt out of, you can early return in your `OnPressed` implementation:

```csharp
public bool OnPressed(KeyBindingPressEvent<MyAction> e)
{
    if (e.Repeat)
        return false;

    if (e.Action == MyAction.Action)
    {
       ...
    }
}
```

# [2021.1106.0](https://github.com/ppy/osu-framework/releases/tag/2021.1106.0)

## `BufferedContainer.CacheDrawnFrameBuffer` has been moved to a constructor argument

```csharp
// old code:
var bufferedContainer = new BufferedContainer() { CacheDrawnFrameBuffer = true };

// new code:
var bufferedContainer = new BufferedContainer(cachedFrameBuffer: true);
```

## `IEffect.CacheDrawnEffect` has been removed

We had no usages of this. If you were using it, nest the effected content in a `BufferedContainer` with `cachedFrameBuffer` set to `true`.

# [2021.1029.0](https://github.com/ppy/osu-framework/releases/tag/2021.1029.0)

## `TextFlowContainer` no longer returns raw `SpriteText`s, returning `ITextPart`s instead

In preparation for adding localisation support to `TextFlowContainer`s, the various `AddText()`/`AddLine()` overloads will no longer return raw `SpriteText`s. Instead, an `ITextPart` structure will be returned.

Via `ITextPart`, the consumer can both access all `Drawables` associated with a given piece of text, as well as react to any future changes in representation of the text by subscribing to `DrawablePartsRecreated`. In the future, with more localisation changes, this event will be invoked once a `LocalisableString`'s displayable content changes, which will trigger a recreation of all parts' drawables, upon which any manual adjustments applied to `Drawables` can be re-applied again.

For consumers wanting to implement their own `ITextPart`s to extend the functionality of `TextFlowContainer`, an abstract `TextPart` is also provided which implements the typical flow of handling `Drawables` and `DrawablePartsRecreated`. The only thing that a consumer has to do when inheriting that class is to implement `CreateDrawablesFor(TextFlowContainer)`.

# [2021.916.1](https://github.com/ppy/osu-framework/releases/tag/2021.916.1)

## `IKeyBindingHandler<T>` and `IScrollBindingHandler<T>` now provide `UIEvent`s

To allow for more arguments without changing the signature of the handling methods, and also for consistency with the input flow in general, both interfaces now provide `UIEvent`s rather than placing each parameter directly on the methods.

```diff
- public bool OnPressed(T action) { }
- public bool OnScroll(T action, float amount, bool isPrecise) { }
- public void OnReleased(T action) { }
+ public bool OnPressed(KeyBindingPressEvent<T> e) { }
+ public bool OnScroll(KeyBindingScrollEvent<T> e) { }
+ public void OnReleased(KeyBindingReleaseEvent<T> e) { }
```

The following regex replacements can be used to simplify migration:
```
Find: bool OnPressed\((\w+)\s+(\w+)\)
Replace: bool OnPressed(KeyBindingPressEvent<$1> e)

Find: void OnReleased\((\w+)\s+(\w+)\)
Replace: void OnReleased(KeyBindingReleaseEvent<$1> e)

Find: bool OnScroll\((\w+) (\w+), float \w+, bool \w+\)
Replace: bool OnScroll(KeyBindingScrollEvent<$1> e)
```

# [2021.907.0](https://github.com/ppy/osu-framework/releases/tag/2021.907.0)

## `AnimationClockComposite.PlaybackPosition` can no longer go below 0

This is not a change to actual playback behaviour - it only affects the return value of the playback position, which now matches the underlying playback behaviour.

# [2021.830.0](https://github.com/ppy/osu-framework/releases/tag/2021.830.0)

## `ITooltip.SetContent` no longer require returning `bool` value

Until now, the method of reusing tooltip instances was problematic (two tooltips handling the same data type could not exist). Instance sharing is now based on the constructed tooltip's `Type`, rather than the data type.

An example of how you should update your code follows.

Before:

```csharp
public override bool SetContent(object content)
{
    if (!(content is CustomContent custom))
        return false;

    text.Text = content.ToString(); // whatever you need to do here.
    return true;
}
```

After:

```csharp
public override void SetContent(object content)
{
    text.Text = content.ToString(); // whatever you need to do here.
}
```

# [2021.818.0](https://github.com/ppy/osu-framework/releases/tag/2021.818.0)

## All custom implementations of `IBindable` must implement `CreateInstance()`

Bindables previously used `Activator.CreateInstance()` to implement the `GetBoundCopy()` call. In profiling this has turned out to be a bottleneck in some scenarios, so in order to reduce the associated runtime overhead to about half, all implementors of `IBindable` now must implement `CreateInstance()`.

This also applies to all inheritors of framework-provided bindable types, who must instead override `CreateInstance()` from their base types.

For a given leaf bindable type in the inheritance hierarchy, the method should return a new constructed instance of the leaf type. The recommended pattern to use is as follows:

```csharp
public class BaseBindable : IBindable
{
    IBindable IBindable.CreateInstance() => CreateInstance();
    protected virtual BaseBindable CreateInstance() => new BaseBindable();

    IBindable IBindable.GetBoundCopy() => GetBoundCopy();
    public BaseBindable GetBoundCopy()
        => IBindable.GetBoundCopyImplementation(this); // automatically calls CreateInstance()
}

public sealed class CustomBindable : BaseBindable
{
    protected override BaseBindable CreateInstance() => new CustomBindable();
}
```

# [2021.803.0](https://github.com/ppy/osu-framework/releases/tag/2021.803.0)

## `EnumLocalisationMapper` for enum localisation has been replaced with per-member `LocalisableDescription` attributes

As the current way for localising `enum`s require a side-class for the mapping, it became a tedious procedure to localise enums and lengthens a lot of files for supporting such.

Therefore a new `LocalisableDescription` attribute has been added allowing for localisation in a more performant and single-lined way.

```diff
-    [LocalisableEnum(typeof(SearchEnumLocalisationMapper))]
     public enum Search
     {
+        [LocalisableDescription(typeof(Strings), nameof(Strings.SearchHide))]
         Hide,
-        Show
-    }
-
-    public class SearchEnumLocalisationMapper : EnumLocalisationMapper<Search>
-    {
-        public override LocalisableString Map(Search value)
-        {
-            switch (value)
-            {
-                case Search.Hide:
-                    return Strings.SearchHide;
-
-                case Search.Show:
-                    return Strings.SearchShow;

-                default:
-                    throw new ArgumentOutOfRangeException(nameof(value), value, null);
-            }
-        }
+        [LocalisableDescription(typeof(Strings), nameof(Strings.SearchShow))]
+        Show
     }
```

This however require from all consumers to store the `LocalisableString`s in static providing classes, similar to [osu!'s `Strings` classes](https://github.com/ppy/osu/tree/master/osu.Game/Localisation).

Benchmarks:
- `EnumLocalisationMapper`:
    |                    Method | Times |          Mean |       Error |       StdDev |        Median |
    |-------------------------- |------ |--------------:|------------:|-------------:|--------------:|
    | GetLocalisableDescription |     1 |      6.866 us |   0.5021 us |     1.383 us |      6.316 us |
    | GetLocalisableDescription |    10 |     93.360 us |   4.9223 us |    13.883 us |     92.744 us |
    | GetLocalisableDescription |   100 |    817.775 us |  31.0268 us |    90.996 us |    812.132 us |
    | GetLocalisableDescription |  1000 | 10,794.319 us | 820.1783 us | 2,392.498 us | 10,718.182 us |

- `LocalisableDescription`:
    |                    Method | Times |         Mean |       Error |      StdDev |       Median |
    |-------------------------- |------ |-------------:|------------:|------------:|-------------:|
    | GetLocalisableDescription |     1 |     4.631 us |   0.1148 us |   0.3331 us |     4.514 us |
    | GetLocalisableDescription |    10 |    54.555 us |   1.3511 us |   3.9412 us |    54.440 us |
    | GetLocalisableDescription |   100 |   540.643 us |  15.9231 us |  45.6864 us |   539.138 us |
    | GetLocalisableDescription |  1000 | 7,218.708 us | 270.7659 us | 768.1179 us | 7,192.596 us |

# [2021.721.0](https://github.com/ppy/osu-framework/releases/tag/2021.721.0)

## `PlatformAction` type is changed to an enum type

If `PlatformAction.ActionType` was matched, change to a matching of `PlatformAction` itself like:

```diff
bool OnPressed(PlatformAction action) {
-    switch (action.ActionType)
+    switch (action)
    {
-        case PlatformActionType.Cut:
+        case PlatformAction.Cut:
```

For `PlatformActionMethod.Delete`, and if only the Delete key should be handled (not Backspace etc.), change to `PlatformAction.Delete`:

```diff
-    switch (action.ActionMethod)
+    switch (action)
    {
-        case PlatformActionMethod.Delete:
+        case PlatformAction.Delete:
```

# [2021.628.0](https://github.com/ppy/osu-framework/releases/tag/2021.628.0)

## `IHasTooltip.Text` is now a `LocalisableString`

`IHasTooltip.Text` now takes a `LocalisableString` instead of  regular string. Users of custom tooltip containers may also have to change checks in the `SetContent()` method of their tooltip type to check if content is a `LocalisableString` instead of a regular string.

## Recently added `GameThread.ThreadPausing` has been removed

This was only added in the previous release, but has since been replaced with the bindable `GameThread.State`, which gives more detail about the current state of the thread.

# [2021.622.0](https://github.com/ppy/osu-framework/releases/tag/2021.622.0)

## `MarkdownHeading.GetFontSizeByLevel` now specifies absolute sizes

The result of this method is now more correctly applied to headers via `FontSize` rather than `Scale`. If you were overriding this method, please multiply your returned values by `20` to maintain sizing compatibility.

## "OpenSans" is no longer provided, with the default font now being "Roboto"

We were already using Roboto in the visual tests contexts, but fallback to OpenSans could be seen for tool windows in both framework and consumer projects. In an effort to consolidate this visually, all framework components now use Roboto, the chosen font for osu!framework design logic.

This means that if you were previously relying on the presence of OpenSans and do not want to switch to Roboto, you will need to re-add the font resources in your project. Instructions on doing this can be found [here](https://github.com/ppy/osu-framework/wiki/Setting-Up-Fonts).

# [2021.419.0](https://github.com/ppy/osu-framework/releases/tag/2021.419.0)

## `RotationDirection.CounterClockwise` has been renamed to `Counterclockwise`

As it appears to be often one word, the enum member `CounterClockwise` has been renamed to `Counterclockwise`.

# [2021.416.0](https://github.com/ppy/osu-framework/releases/tag/2021.416.0)

## `IBindable`, `IBindable<T>`, and `IBindableList<>` are no longer `IParseable`

`IParseable` is only implemented on the `Bindable<>` and `BindableList<>` classes. Code that relied on this functionality should instead cast to `IParseable`.

# [2021.330.0](https://github.com/ppy/osu-framework/releases/tag/2021.330.0)

## InputHandler no longer has a Priority property

Priority is now decided by the construction order of `InputHandler`s in `CreateAvailableInputHandlers`.

# [2021.317.0](https://github.com/ppy/osu-framework/releases/tag/2021.317.0)

## ConfigManager.Set is now protected and renamed to `SetDefault`

Previously this method was `public` for convenience, but as it implicitly set the `Default` value of bindables it touched, could cause unexpected behaviour.

- For initialising defaults, switch to using `SetDefault` from `InitialiseDefaults`
- For changing configuration values externally, use `SetValue` instead of `Set`

```csharp
// previously
config.Set(ConfigType.Name, true);

// now
config.SetValue(ConfigType.Name, true);
```

## osuTK support has been removed in most places

If you were supporting it via `GameHost` initialisation, you may need to remove a parameter. Internally, we still support it in a minimal way for Xamarin platforms.

# [2021.225.0](https://github.com/ppy/osu-framework/releases/tag/2021.225.0)

## The regular weight of OpenSans has been renamed to OpenSans-Regular

Until now this font wasn't following the convention we use everywhere else. If you are referencing it directly, please update your strings to point to the new name.

## LocalisedString no longer exists

For strings which have romanisable content, `RomanisableString` should be used instead.

## Multiple UI Components now use `LocalisableString` in place of `string`

If you have custom implementations, you will need to update the overridden signatures.

## Explicit `ToString` may be required where previous not

Getting the current value of some strings (ie. on UI Components) will now require an explicit call to `.ToString()`.

# [2021.106.0](https://github.com/ppy/osu-framework/releases/tag/2021.106.0)

## Games will now throw (and crash) immediately on performing cross-thread transform operations

This may mean that as a framework consumer you need to re-think how you are performing operations between drawables. The easiest way to avoid issue is to use `Schedule` to force the target code to be on the correct thread.

# [2020.1009.0](https://github.com/ppy/osu-framework/releases/tag/2020.1009.0)

## `WaveformGraph.BaseColour` should be used instead of `Colour` for the default frequency colour

`Colour` now uniformly affects the entire graph as expected.

# [2020.901.0](https://github.com/ppy/osu-framework/releases/tag/2020.901.0)

## `IAggregateAudioAdjustment.GetAggregate()` has been made an extension method

To avoid aggregate adjustment implementations from having to implement both `Aggregate{Volume,Balance,Frequency,Tempo}` and `GetAggregate()`, the latter has been made an extension method returning one of the aforementioned four values for the appropriate property of the adjustment.

Possible compilation failures related to `GetAggregate()` should be resolved by adding a using statement for the `osu.Framework.Audio` namespace to the files affected. 

# [2020.819.0](https://github.com/ppy/osu-framework/releases/tag/2020.819.0)

## `TextBox` events only trigger on user input

The events were specifically exposed to provide feedback on user input, but were triggering on programmatic manipulation of the textbox's text. See https://github.com/ppy/osu-framework/pull/3839/files.

# [2020.710.0](https://github.com/ppy/osu-framework/releases/tag/2020.710.0)

## `TextBox` will no longer trigger sound effects

This is a fringe use-case, as the samples were never included with the framework, but if you happened to be providing them in your game resources, you will need to update your `TextBox` implementation to trigger them again. Check the [osu!-side changes](https://github.com/ppy/osu/pull/9216) for an example of how to do this.

## `CachedModelDependencyContainer` models must now only contain read-only fields

Mutating bindables of models attached to a `CachedModelDependencyContainer` can lead to crashes or otherwise unexpected behaviour, and is now disallowed.

# [2020.302.0](https://github.com/ppy/osu-framework/releases/tag/2020.302.0)

## `FrameworkDebugConfig` no longer exposes `PerformanceLogging`

Performance logging is now toggled by opening "Global Statistics" overlay via Ctrl+F2. To manually toggle it, you may access `GameHost.PerformanceLogging`, but note that a change to this bindable will be overridden by toggling the statistics overlay.

## Layout validation and invalidation has been reworked

To better cover edge-case scenarios where layout wasn't properly refreshed, the process of layout invalidation has changed.

### In cases where `Invalidate()` affected a `Cached` member, the following adjustment is necessary:

```diff
public class MyDrawable : Drawable
{
-    private readonly Cached myLayoutValue = new Cached();
-
-    protected override bool OnInvalidate(Invalidation invalidation, Drawable source, bool shallPropagate)
-    {
-        var result = base.OnInvalidate(invalidation, source, shallPropagate);
-
-        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
-            result &= !myLayoutValue.Invalidate();
-
-        return result;
-    }

+    /* can also be a LayoutValue<T> */
+    private readonly LayoutValue myLayoutValue = new LayoutValue(Invalidation.DrawSize | Invalidation.Presence);
+
+    public MyDrawable()
+    {
+        AddLayout(myLayoutValue);
+    }
}
```

### In cases where `InvalidateFromChild()` affected a `Cached` member, the following adjustment is necessary:

```diff
public class MyDrawable : Drawable
{
-    private readonly Cached myLayoutValue = new Cached();
-
-    protected override void InvalidateFromChild(Invalidation invalidation, Drawable source)
-    {
-        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
-            myLayoutValue.Invalidate();
-    }

+    /* can also be a LayoutValue<T> */
+    private readonly LayoutValue myLayoutValue = new LayoutValue(Invalidation.DrawSize | Invalidation.Presence, InvalidationSource.Child);
+
+    public MyDrawable()
+    {
+        AddLayout(myLayoutValue);
+    }
}
```

### In cases where _both_ `Invalidate()` and `InvalidateFromChild()` affected the same `Cached` member, the following adjustment is required:

```diff
public class MyDrawable : Drawable
{
-    private readonly Cached myLayoutValue = new Cached();
-
-    protected override bool OnInvalidate(Invalidation invalidation, Drawable source, bool shallPropagate)
-    {
-        var result = base.OnInvalidate(invalidation, source, shallPropagate);
-
-        if ((invalidation & Invalidation.DrawSize) > 0) /* any invalidation type of your choice */
-            result &= !myLayoutValue.Invalidate();
-
-        return result;
-    }
-
-    protected override void InvalidateFromChild(Invalidation invalidation, Drawable source)
-    {
-        if ((invalidation & Invalidation.Presence) > 0) /* any invalidation type of your choice */
-            myLayoutValue.Invalidate();
-    }

+    /* can also be a LayoutValue<T> */
+    private readonly LayoutValue myLocalLayoutValue = new LayoutValue(Invalidation.DrawSize);
+    private readonly LayoutValue myChildLayoutValue = new LayoutValue(Invalidation.Presence, InvalidationSource.Child);
+
+    public MyDrawable()
+    {
+        AddLayout(myLocalLayoutValue);
+        AddLayout(myChildLayoutValue);
+    }
+
+    protected override void Update()
+    {
+        base.Update();
+
+        /* wherever your re-validation was previously done */
+        if (!myChildLayoutValue.IsValid)
+        {
+            myLocalLayoutValue.Invalidate();
+            myChildLayoutValue.Validate();
+        }
+
+        if (!myLocalLayoutValue.IsValid)
+        {
+            /* your custom validation */
+            myLocalLayoutValue.Validate();
+        }
+    }
}
```

### In cases where custom logic that can't be described as layout was done in `Invalidate()`, the following adjustment is necessary:

```diff
public class MyDrawable : Drawable
{
-    protected override bool OnInvalidate(Invalidation invalidation, Drawable source, bool shallPropagate)
-    {
-        var result = base.OnInvalidate(invalidation, source, shallPropagate);
-
-        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
-        {
-            performCustomAction();
-            result = true;
-        }
-
-        return result;
-    }

+    protected override bool OnInvalidate(Invalidation invalidation, InvalidationSource source)
+    {
+        var result = base.OnInvalidate(invalidation, source);
+
+        if ((invalidation & (Invalidation.DrawSize | Invalidation.Presence)) > 0) /* any invalidation type of your choice */
+        {
+            performCustomAction();
+            result = true;
+        }
+
+        return result;
+    }

    private void performCustomAction()
    {
        /* custom invalidation logic here */
    }
}
```

# [2020.218.0](https://github.com/ppy/osu-framework/releases/tag/2020.218.0)

## `BindableList<T>.ItemsAdded` is now obsolete

The `ItemsAdded` event failed to provide enough context when items are moved around or inserted into the list.

It has been replaced by the `CollectionChanged` [NotifyCollectionChangedEventHandler](https://docs.microsoft.com/en-au/dotnet/api/system.collections.specialized.notifycollectionchangedeventhandler?view=netframework-4.8), which provides context such as the newly-added items and the indices at which the addition took place.

This event is triggered by the following methods with `Action = NotifyCollectionChangedAction.Add`:
```csharp
BindableList<T>.Add(T item)
BindableList<T>.Insert(int index, T item)
BindableList<T>.AddRange(IEnumerable<T> items)
```

The following is an example migration utilising the new `CollectionChanged` event:
```diff
BindableList<int> list = new BindableList<int>();

- list.ItemsAdded += items =>
- {
-     foreach (var item in items)
-         Console.WriteLine($"Added: {item}");
- }

+ list.CollectionChanged += (_, args) =>
+ {
+     switch (args.Action)
+     {
+         case NotifyCollectionChangedAction.Add:
+             foreach (var item in args.NewItems.Cast<int>())
+                 Console.WriteLine($"Added: {item});
+             break;
+     }
+ }
```

Note that `CollectionChanged` should _not_ be used in conjunction with the `ItemsAdded` event.

## `BindableList<T>.ItemsRemoved` is now obsolete

As above, the `ItemsRemoved` revent has also been replaced by the `CollectionChanged` [NotifyCollectionChangedEventHandler](https://docs.microsoft.com/en-au/dotnet/api/system.collections.specialized.notifycollectionchangedeventhandler?view=netframework-4.8).

This event is triggered by the following methods with `Action = NotifyCollectionChangedAction.Remove`:
```csharp
BindableList<T>.Clear()
BindableList<T>.Remove(T item)
BindableList<T>.RemoveRange(int index, int count)
BindableList<T>.RemoveAt(int index)
BindableList<T>.RemoveAll(Predicate<T> match)
```

The following is an example migration utilising the new `CollectionChanged` event:
```diff
BindableList<int> list = new BindableList<int>();

- list.ItemsRemoved += items =>
- {
-     foreach (var item in items)
-         Console.WriteLine($"Removed: {item}");
- }

+ list.CollectionChanged += (_, args) =>
+ {
+     switch (args.Action)
+     {
+         // Handles both clear and remove events
+         case NotifyCollectionChangedAction.Remove:
+             foreach (var item in args.OldItems.Cast<int>())
+                 Console.WriteLine($"Removed: {item});
+             break;
+     }
+ }
```

# [2020.122.0](https://github.com/ppy/osu-framework/releases/tag/2020.122.0)

## InputManager.CreateButtonManagerFor was renamed to InputManager.CreateButtonEventManagerFor

Changed to match other methods and the class name it was creating.

## Various input "end" events now return `void` and can no longer block propagation

Events affected:
```csharp
Drawable.OnDoubleClick()
Drawable.OnDrag()
Drawable.OnDragEnd()
Drawable.OnMouseUp()
Drawable.OnKeyUp()
Drawable.OnJoystickRelease()
IKeyBindingHandler<T>.OnReleased()
```

A drawable may return `false` for an input "begin" event to allow the event to propagate further through the hierarchy. Previously, such a drawable could then return `true` for the input "end" event and prevent the event from propagating to the drawables which the input "begin" event was propagated to.  
This could lead to incorrect implementations where drawables are left in weird states after handling input events.

By returning `void`, the input "end" events can no longer be blocked by other drawables in the hierarchy and are guaranteed to be invoked if their respective input "begin" event was previously invoked.

The following table illustrates the events for which the relationship is satisfied:

|     begin event     |        end event(s)       |
| ------------------- | ------------------------- |
| `OnDragStart()`     | `OnDrag()`, `OnDragEnd()` |
| `OnMouseDown()`     | `OnMouseUp()`             |
| `OnKeyDown()`       | `OnKeyUp()`               |
| `OnJoystickPress()` | `OnJoystickRelease()`     |
| `OnPressed()`       | `OnReleased()`            |

# [2020.109.0](https://github.com/ppy/osu-framework/releases/tag/2020.109.0)

## The namespace `osu.Framework.MathUtils` has been removed

All existing utility classes have been moved to the namespace `osu.Framework.Utils`.

# [2019.1224.0](https://github.com/ppy/osu-framework/releases/tag/2019.1224.0)

## `TextBox` and all related components are now abstract

`BasicTextBox` is provided as a drop-in replacement that comes with the framework design language.

## `PasswordTextBox` has been renamed 

Renamed to `BasicPasswordTextBox` and comes with the framework design language.

# [2019.1211.1](https://github.com/ppy/osu-framework/releases/tag/2019.1211.1)

## The value provided in the constructor for `Bindable<T>` is now used as both the initial and default value

The following lines of code are now identical.
```csharp
var bindable1 = new Bindable<int>(10) { Default = 10 };
var bindable2 = new Bindable<int>(10);
```

# [2019.1210.1](https://github.com/ppy/osu-framework/releases/tag/2019.1210.1)

## PerformanceLogging was moved to DebugConfig

As seen in https://github.com/ppy/osu/issues/6795, some users are turning on performance logging and leaving it on. This is not intended, as it adds a noticeable overhead to retrieve and write the stack traces to disk.

This change allows the setting to reset each game execution, rather than be saved to a user's configuration file.

# [2019.1121.0](https://github.com/ppy/osu-framework/releases/tag/2019.1121.0)

## .NET Standard 2.1

osu!framework now targets .NET Standard 2.1. Consumers of the framework will need to install the [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) and change their projects to target `netstandard2.1`/`netcoreapp3.0`.

## `WebRequest.ResponseString` and `WebRequest.ResponseData` properties were converted to methods

`GetResponseString` and `GetResponseData` methods are provided as a replacement. This change was done in order to indicate that these operations are expensive(they do memory allocations in the heap on each call).

# [2019.1104.0](https://github.com/ppy/osu-framework/releases/tag/2019.1104.0)

## `Button` has been made abstract

`BasicButton` is provided as a drop-in replacement that comes with the framework design language.

# [2019.921.0](https://github.com/ppy/osu-framework/releases/tag/2019.921.0)

## `Quad.ConservativeArea` has been removed

It was inaccurate for certain `Quad`s. Use `Area` as a replacement.

# [2019.821.0](https://github.com/ppy/osu-framework/releases/tag/2019.821.0)

## `SpriteText.UseFixedWidthForCharacter` has been removed

Provide an array of `char`s as `SpriteText.FixedWidthExcludeCharacters` instead.

## `SpriteText.GetTextureForCharacter` and `SpriteText.GetFallbackTextureForCharacter` have been removed.

Glyphs must now always be retrieved through an `ITexturedGlyphLookupStore`. The `SpriteText.CreateTextBuilder()` method is provided to allow overriding the store which glyphs are retrieved from.

## `BlendingModes` are now obsoleted

Please switch to using `BlendingParameters` static properties instead, whcih provide the same functionality. This change was made to expose full control over blend equations and simplify the blending class structure, which previously spanned three related types.

# [2019.809.0](https://github.com/ppy/osu-framework/releases/tag/2019.809.0)

## `Cached` is now a class and requires object initialisation

Usages of `Cached` without initialising (via `new Cached()`) will need to be updated.

## `Animation` no longer supports `AutoSizeAxes`

`Animation` will automatically auto-size on axes which are not relative sized (via `RelativeSizeAxes`) and for which `Size` has not been set.

# [2019.726.0](https://github.com/ppy/osu-framework/releases/tag/2019.726.0)

## `Path` now supports `AutoSizeAxes` and is set to auto-size in both axes by default

There are now three modes of operation for paths:
```
// Auto size
new Path()

// Static size
new Path
{
    AutoSizeAxes = Axes.None,
    Size = new Vector2(100)
}

// Relative size
new Container
{
    Size = new Vector2(100),
    new Path
    {
        AutoSizeAxes = Axes.None,
        RelativeSizeAxes = Axes.Both
    }
}
```

# [2019.702.0](https://github.com/ppy/osu-framework/releases/tag/2019.702.0)

## `InputKey` enum names for extra mouse buttons have been renamed.

```
MouseButtonX -> ExtraMouseButtonX
```

# [2019.628.0](https://github.com/ppy/osu-framework/releases/tag/2019.628.0)

## `Menu` and several related components have been made abstract

For each component, there are two ways to resolve resulting errors:

1. `ContextMenuContainer`
    1. `BasicContextMenuContainer` is provided as a drop-in replacement that comes with the framework design language.
    2. Implement your own local `ContextMenuContainer`. The following is consumable code that matches the removed implementation. Rename it to suit your project and update your references:  
        ```csharp
        public class MyContextMenuContainer : ContextMenuContainer
        {
            // If you have a custom menu, provide it here.
            protected override Menu CreateMenu() => new BasicMenu(Direction.Vertical);
        }
        ```
2. `Menu`
    1. `BasicMenu` is provided as a drop-in replacement that comes with the framework design language.
    2. Implement your own local `Menu`. The following is consumable code that matches the removed implementation. Rename it to suit your project and update your references:  
        ```csharp
        public class MyMenu : Menu
        {
            public MyMenu(Direction direction, bool topLevelMenu = false)
                : base(direction, topLevelMenu)
            {
            }
        
            protected override Menu CreateSubMenu() => new MyMenu(Direction.Vertical);
        
            protected override DrawableMenuItem CreateDrawableMenuItem(MenuItem item) => new MyDrawableMenuItem(item);
        
            // If you have a custom scroll container, provide it here.
            protected override ScrollContainer<Drawable> CreateScrollContainer(Direction direction) => new BasicScrollContainer(direction);
        
            private class MyDrawableMenuItem : DrawableMenuItem
            {
                public MyDrawableMenuItem(MenuItem item)
                    : base(item)
                {
                }
        
                protected override Drawable CreateContent() => new SpriteText
                {
                    Anchor = Anchor.CentreLeft,
                    Origin = Anchor.CentreLeft,
                    Padding = new MarginPadding(5),
                    Font = new FontUsage(size: 17),
                };
            }
        }
        ```
3. `DropdownMenu` / `DropdownMenuItem`. This is only applicable if you already implement a custom `Dropdown<T>`.
    1. The following is consumable code that matches the removed implementation. Rename it to suit your project and update your references:
        ```csharp
        public class MyDropdown<T> : Dropdown<T>
        {
            // If you have a custom header, provide it here.
            protected override DropdownHeader CreateHeader() => null;
        
            protected override DropdownMenu CreateMenu() => new MyDropdownMenu();
        
            private class MyDropdownMenu : DropdownMenu
            {
                // If you have a custom menu, provide it here.
                protected override Menu CreateSubMenu() => new BasicMenu(Direction);
        
                // If you have a custom scroll container, provide it here.
                protected override ScrollContainer<Drawable> CreateScrollContainer(Direction direction) => new BasicScrollContainer(direction);
        
                protected override DrawableDropdownMenuItem CreateDrawableDropdownMenuItem(MenuItem item) => new MyDrawableDropdownMenuItem(item);
        
                private class MyDrawableDropdownMenuItem : DrawableDropdownMenuItem
                {
                    public MyDrawableDropdownMenuItem(MenuItem item)
                        : base(item)
                    {
                    }
        
                    // If you have custom content, provide it here.
                    protected override Drawable CreateContent() => new SpriteText
                    {
                        Anchor = Anchor.CentreLeft,
                        Origin = Anchor.CentreLeft,
                        Padding = new MarginPadding(5),
                        Font = new FontUsage(size: 17),
                    };
                }
            }
        }
        ```

## `DebugSetting.ActiveGCMode` no longer exists

If you were manually setting this, you can do so via [.NET methods](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/latency?view=netframework-4.8).

# [2019.614.0](https://github.com/ppy/osu-framework/releases/tag/2019.614.0)

## `ScrollContainer<T>` is now abstract (and `ScrollContainer` no longer exists)

In a push to make the provided framework components design agnostic, `ScrollContainer` is no longer provided. As a framework consumer you have a few options to resolve this:

- Use `BasicScrollContainer`

The framework now provides `BasicScrollContainer` and `BasicScrollContainer<T>`. These are drop-in replacements but come with the framework design language, so it is only recommended as a temporary solution.

- Implement your own local `ScrollContainer`

Here is consumable code to provide a local implementation that matches the removed one. Rename to suit your project and update your references:

```csharp
public class MyScrollContainer : ScrollContainer<Drawable>
{
    public MyScrollContainer(Direction scrollDirection = Direction.Vertical)
        : base(scrollDirection)
    {
    }

    protected override ScrollbarContainer CreateScrollbar(Direction direction) => new MyScrollbar(direction);

    protected class MyScrollbar : ScrollbarContainer
    {
        private const float dim_size = 10;

        private readonly Color4 hoverColour = Color4.White;
        private readonly Color4 defaultColour = Color4.Gray;
        private readonly Color4 highlightColour = Color4.Black;

        private readonly Box box;

        public MyScrollbar(Direction scrollDir)
            : base(scrollDir)
        {
            Colour = defaultColour;

            Blending = BlendingMode.Additive;

            CornerRadius = 5;

            const float margin = 3;

            Margin = new MarginPadding
            {
                Left = scrollDir == Direction.Vertical ? margin : 0,
                Right = scrollDir == Direction.Vertical ? margin : 0,
                Top = scrollDir == Direction.Horizontal ? margin : 0,
                Bottom = scrollDir == Direction.Horizontal ? margin : 0,
            };

            Masking = true;
            Child = box = new Box { RelativeSizeAxes = Axes.Both };

            ResizeTo(1);
        }

        public override void ResizeTo(float val, int duration = 0, Easing easing = Easing.None)
        {
            Vector2 size = new Vector2(dim_size)
            {
                [(int)ScrollDirection] = val
            };
            this.ResizeTo(size, duration, easing);
        }

        protected override bool OnHover(HoverEvent e)
        {
            this.FadeColour(hoverColour, 100);
            return true;
        }

        protected override void OnHoverLost(HoverLostEvent e)
        {
            this.FadeColour(defaultColour, 100);
        }

        protected override bool OnMouseDown(MouseDownEvent e)
        {
            if (!base.OnMouseDown(e)) return false;

            //note that we are changing the colour of the box here as to not interfere with the hover effect.
            box.FadeColour(highlightColour, 100);
            return true;
        }

        protected override bool OnMouseUp(MouseUpEvent e)
        {
            if (e.Button != MouseButton.Left) return false;

            box.FadeColour(Color4.White, 100);

            return base.OnMouseUp(e);
        }
    }
}
```

# [2019.611.0](https://github.com/ppy/osu-framework/releases/tag/2019.611.0)

## `LinearVertexBuffer` and `QuadVertexBuffer` can no longer be constructed outside of osu!framework.

For existing code, `LinearBatch(size, 1)` and `QuadBatch(size, 1)` may be used to replace vertex buffer usages. Code must be updated to always invoke `batch.Add(vertex)` with the vertices that should be drawn.

`VertexBuffer` may still be derived for custom implementations.

## VisibilityContainer now exposes visibility via a bindable

`VisibilityContainer` no longer implements `IStateful<Visibility>`, instead exposing a single `Bindable<Visibility> State`.

# [2019.606.0](https://github.com/ppy/osu-framework/releases/tag/2019.606.0)

## `Texture.DrawTriangle()` and `Texture.DrawQuad()` have been removed ([#2475](https://github.com/ppy/osu-framework/pull/2475))

Use `DrawNode.DrawTriangle()` and `DrawNode.DrawQuad()` instead.

# [2019.604.0](https://github.com/ppy/osu-framework/releases/tag/2019.604.0)

## Global `TrackStore` and `ResourceStore`s can no longer access resources that are out of their respective namespaces.

Previously, it was possible to access any resource as a part of `TrackStore` and `SampleStore` via their `Get` methods because the entire resource store would be nested inside them. 

This nesting has been removed, so you can now only access files that are part of their specified namespaces (folders). 

* For `TrackStore`, this will be the `Tracks` namespace. 
* For `SampleStore`, this will be the `Samples` namespace.

## Audio stores have been renamed

To bring naming in-line with the purpose of audio stores, the following classes have been renamed:

`SampleManager` -> `SampleStore`  
`TrackManager` -> `TrackStore`

In line with this, return values of `GetSampleStore` and `GetTrackStore` now return `ISampleStore` and `ITrackStore` respectively, hiding the underlying manager logic.

The public variables relating to these stores have been renamed to reflect this change:

`AudioManager.Track` -> `AudioManager.Tracks`  
`AudioManager.Sample` -> `AudioManager.Samples`

## Virtual tracks must be retrieved from `ITrackStore`

`ITrackStore` now has a `GetVirtual` method which will create a virtual track. This will handle adding the track correctly to the audio subsystem. If one wishes to create a custom type of virtual tracks (a super-edge-case) you could implement the `ITrackStore` interface and manually add your component to `AudioManager` via `AddItem`.

# [2019.523.0](https://github.com/ppy/osu-framework/releases/tag/2019.523.0)

## TabControl can now select nothing [#2430](https://github.com/ppy/osu-framework/pull/2430)

While this does not match most OS implementations of a tab control, this was deemed useful for o!f usage scenarios.

Previously, this code would throw an exception.

```csharp
tabControl.Current.Value = null;
```

Now it is allowed.

## Dropdown can now select nothing [#2428](https://github.com/ppy/osu-framework/pull/2428)

Previously, this code would throw an exception.

```csharp
dropdownMenu.Current.Value = null;
```

Now it is allowed. Note that implementations of dropdown should be updated to handle this (common scenario is that the `DropdownHeader` would not correctly handle a `string.Empty` case if it was using AutoSize in the Y axis).

## `TabItem.IsRemovable` is true by default [#2425](https://github.com/ppy/osu-framework/pull/2425)

This felt like a more sane default. Any existing usage of custom `TabItem`s where `IsRemovable` was overridden, or any existing usage where removal is explicitly *not* wanted need to be updated.

# [2019.514.0](https://github.com/ppy/osu-framework/releases/tag/2019.514.0)

## `Game.FrameStatisticsMode` is now a bindable named `Game.FrameStatistics` [#2399](https://github.com/ppy/osu-framework/pull/2399)

This allows consumers to hook value changed events so they can, for instance, save the (framework) FPS display's state to config and handle hotkey-based changes.

## Visual `TestCase`s are now `TestScene`s [#2365](https://github.com/ppy/osu-framework/pull/2365)

The term `TestCase` is used by testing frameworks to denote single methods inside a test class. We were using it on the class itself as a prefix, which got quite confusing. This resolves the conflict and feels more correct as these are visual tests – ie. "scenes".

Note that the `TestCase` prefix is still supported by tooling for the time being. We still recommend you update (via a simple rename) as soon as possible.

# [2019.427.0](https://github.com/ppy/osu-framework/releases/tag/2019.427.0)

## `Drawable.ApplyDrawNode()` has been removed [#2314](https://github.com/ppy/osu-framework/pull/2314)

The direction of application of `DrawNode` states has been inversed.

Previously:
```csharp
class MyCustomDrawable : Drawable
{
    private bool state;

    protected override DrawNode CreateDrawNode() => new CustomDrawNode();

    protected override void ApplyDrawNode(DrawNode node)
    {
        base.ApplyDrawNode(node);

        var n = (CustomDrawNode)n;
        
        n.State = state; 
    }

    private class CustomDrawNode : DrawNode
    {
        public bool State;
    }
}
```

Now:
```csharp
class MyCustomDrawable : Drawable
{
    private bool state;

    protected override DrawNode CreateDrawNode() => new CustomDrawNode(this);

    private class CustomDrawNode : DrawNode
    {
        protected new MyCustomDrawable Source => (MyCustomDrawable)base.Source;

        private bool state;

        public CustomDrawNode(MyCustomDrawable source)
            : base(source)
        {
        }

        public override void ApplyState()
        {
            base.ApplyState();

            state = Source.state;
        }
    }
}
```

## The namespace of `EdgeEffectParameters` and `EdgeEffectType` has changed [#2314](https://github.com/ppy/osu-framework/pull/2314)

They now reside in `osu.Framework.Graphics.Effects`.

# [2019.327.0](https://github.com/ppy/osu-framework/releases/tag/2019.327.0)

## Font fallback order changes [#2296](https://github.com/ppy/osu-framework/pull/2296)

Games adding their own fonts can now do so directly to `Game.Fonts`. Fallback (framework-side) fonts are now always at the lowest priority.

Previously:
```csharp
// this completely overrides the framework default. will need to change once we make a proper FontStore.
dependencies.Cache(Fonts = new FontStore(new GlyphStore(Resources, @"Fonts/FontAwesome")));
Fonts.AddStore(new GlyphStore(Resources, @"Fonts/osuFont"));
```

now:
```csharp
Fonts.AddStore(new GlyphStore(Resources, @"Fonts/osuFont"));
```

## FontAwesome and SpriteIcon is provided by the framework [#2293](https://github.com/ppy/osu-framework/pull/2293)

If you manually added either of these to your project, please remove them.

Usage:
```csharp
new SpriteIcon
{
    Icon = FontAwesome.Refresh,
};
```

# [2019.319.0](https://github.com/ppy/osu-framework/releases/tag/2019.319.0)

## Parameter order of `AddUntilStep` / `AddWaitStep` reversed [#2256](https://github.com/ppy/osu-framework/pull/2256)

In order to standardise parameter order, these two methods' parameters have been reversed. Old parameter order methods are available temporarily (as `[Obsolete]`) to ease migration.

## New attribute `[SetUpSteps]` added [#2266](https://github.com/ppy/osu-framework/pull/2266)

Finally allows steps to be added that will run before every `[Test]` method.

# [2019.221.0](https://github.com/ppy/osu-framework/releases/tag/2019.221.0)

## Implicit operator removed from `Bindable` [#2152](https://github.com/ppy/osu-framework/pull/2152)

In order to avoid accidental misuse / misunderstandings, `.Value` must always be added to `Bindable` when requesting its value. One such case which used to be confusing:

```csharp
Bindable<Drawable> bindableDrawable = new Bindable<Drawable>();

if (bindableDrawable != null)
{
    // true
}

if (bindableDrawable.Value != null)
{
    // false
}

Drawable drawable = null;

if (bindableDrawable == drawable)
{
    // ???
}
```

### Bindable.ValueChanged now provides `ValueChangedEvent` [#2012](https://github.com/ppy/osu-framework/pull/2012)

It is now possible to access the old value. This was a common requirement in bindable usage which we had issues with until now.

```csharp

bindable.ValueChanged += e =>
{
    Console.WriteLine($"Value changed from {e.OldValue} to {e.NewValue}");
}
```

### Introduction of `FontUsage` [#2043](https://github.com/ppy/osu-framework/pull/2043)

While old methods of setting font attributes will still work, they are now marked as `[Obsoleted]`. You should use `Font = new FontUsage(size: 200)` going forward. For convenience, you can adjust existing fonts like so:

```csharp
var font = new FontUsage(family: "SourceCodePro");

var fontWithSize = font.With(size: 24);
```
