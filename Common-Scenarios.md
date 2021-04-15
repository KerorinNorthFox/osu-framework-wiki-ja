This page intends to contains a bunch of common scenarios which users may run into while using the framework, which may have straightforward but not immediately obvious answers. Each entry should be as brief as possible, and include a code sample where applicable.

## `GridContainer` where earlier cells handle input before later cells

This can be achieved by setting `Depth` on cell content. Constructing a later cell with higher depth than a preceding one will allow it to handle input earlier.

## Creating a rounded line with end caps outside the size of the filled portion

![20210414 181003 (dotnet)](https://user-images.githubusercontent.com/191335/114685263-acf8e480-9d4c-11eb-9315-440e70d4ffa6.png)

```csharp
public class RoundedLine : CompositeDrawable
{
    private readonly Circle content;

    public override Quad ScreenSpaceDrawQuad => content.ScreenSpaceDrawQuad;

    public ExtendableCircle()
    {
        Padding = new MarginPadding { Horizontal = -circle_size / 2f };
        InternalChild = content = new Circle
        {
            RelativeSizeAxes = Axes.Both
        };
    }
}
```