# Graphics Coordinate and Layout Systems

Once added to a parent `Container` hierarchy, `Drawable` objects will adopt the pixel coordinate space of that container. By default, all positioning and sizing is static via absolute pixel values regardless of the container size. That being said, a variety of mechanisms exist to allow relative sizing in order to properly scale to different window sizes and device screens.

In order to help automate laying elements out in common positions, `Drawable` implements 2 properties that will help  determine its initial position.

* `Anchor` will determine its origin’s position relative to the parent container. `Centre` will place the element in the middle of the container, whereas other options, such as `TopLeft` will position it along the outer edges of the container.
* `Origin` will determine the offset of the element’s content from its own origin. For example `Centre` will mean the origin of the element will be located in the middle of its content. `TopLeft` will make the origin the top left corner of the content.

```csharp
box = new Box
{
	Anchor = Anchor.TopLeft,
	Origin = Anchor.TopLeft
}
```

Once these two properties have been set, they can be further customized with additional properties.

* Setting `Position` on a `Drawable` will offset it from its initial position. This supports both positive and negative values to offset the element in any direction.
* Setting `Size` will change the size, in pixels of the width and height of the element. The positioning of the element will be updated around its `Anchor` and `Origin` values factoring in its new size.

```csharp
box.Position = new Vector2(100.0f, 50.0f);
box.Size = new Vector2(400.0f, 400.0f);
```

# Relative Layout and Sizing

For cases where the size of the container is flexible, it is possible to override the behavior of `Drawable` to instead rely on relative values, instead of absolute.

To enable this behaviour, access the `RelativePositionAxes` and `RelativeSizeAxes` enum properties, and set which axes you would to be relative.

Once enabled, instead of absolute pixel values, you can set values between `0.0f` and `1.0f` to denote the relative value. In this case `0.0f` represents 0% of the parent container, and `1.0f` represents 100%.

```csharp
box.RelativePositionAxes = Axes.Both;
box.Position = new Vector2(0.2f, 0.5f);
```

# Scaling Absolute Values to Screen Size

In most cases, working with relative values is much more vague and difficult to work in as opposed to absolute values.

In order to allow absolute positioning whilst still allowing flexible screen sizes, osu-framework provides a type of `Container` called `DrawSizePreservingFillContainer`.

```csharp
new DrawSizePreservingFillContainer
{
    Child = box = new Box
    {
        Anchor = Anchor.TopLeft,
        Origin = Anchor.TopLeft,
        Colour = Color4.Orange,
        Size = new Vector2(200),
    }
};
```

This type of container implements a `TargetDrawSize` property (Default value is `Vector2(1024, 768)`) which represents the size of its canvas. All children can then be sized in relation to this container size.

When the parent container is resized, `DrawSizePreservingFillContainer` will resize to fill that parent, but will also properly upscale or downscale all of the child `Drawable` elements to fill the same space.

By default, `DrawSizePreservingFillContainer` will scale all of its child content while preserving the content’s aspect ratio using a ‘scale to fit’ strategy. Depending on your content, and the size of the screens you plan to support, the `Strategy` enum allows for other types of scaling as well:

* `Minimum` - The aspect ratio scale is determined by the smaller dimension value of the container’s current size. This results in all of the content being scaled down to fit into the container with no overflow. This is the default value
* `Maximum` - The aspect ratio scale is based off the larger dimension value of the container’s current size. This results in content being scaled up to fill the available boundaries and *may* result in some content getting clipped.
* `Average` - The average value between the minimum and the maximum scales is calculated and applied. This is a good compromise between the both previous values.
* `Separate` - No aspect ratio is applied at all. All elements will stretch and distort relative to the container’s dimensions.