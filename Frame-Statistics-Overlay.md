# Frame Statistics

osu! framework provides in-picture debugging overlays, defined in `osu.Framework.Input.FrameworkActionContainer`. One such overlay is the Frame Statistics Overlay.

## Overview
**Control-F11** toggles the frame statistics overlay. The first time you toggle this overlay, you will be greeted with a minimized version of it. This view displays the frame timings of the four main gameplay threads: Audio, Input, Update, and Draw. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538264876561203200/Screen_Shot_2019-01-25_at_3.31.08_PM.png)

**Pressing Control-F11 again** will expand the overlay, displaying a graph of the activity on each thread. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538266387014221824/Screen_Shot_2019-01-25_at_4.58.06_PM.png)

**Holding down the control key** in this state will expand the graph further, allowing you to peek at how many events of each type are occuring on each thread.

![](https://cdn.discordapp.com/attachments/318886668889227266/538268818951241754/unknown-2_copy.png)

## Statistics

The statistics for this overlay are defined in `osu.Framework.Statistics.FrameStatistics`. 

* [Invalidations](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Caching/Cached.cs#L73): The amount of Invalidate() calls performed on [cached](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Caching/Cached.cs) objects.
* [Refreshes](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Caching/Cached.cs#L83): The amount of Validate() calls performed on [cached](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Caching/Cached.cs) objects.
* [DrawNodeCtor](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Drawable.cs#L1691): The amount of new [DrawNodes](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/DrawNode.cs) created.
* [DrawNodeAppl](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Drawable.cs#L1697): The amount of [DrawNodes](https://github.com/ppy/osu-framework/blob/naster/osu.Framework/Graphics/DrawNode.cs) updated due to a disagreement between the [Drawable's](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/Drawable.cs) Invalidation ID and the node's Invalidation ID.
* [ScheduleInvk](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Containers/CompositeDrawable.cs#L721): The amount of scheduled tasks not yet performed.
* [InputQueue](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/InputManager.cs#L288): The amount of [UserInputManagers](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/UserInputManager.cs) handling non-positional (keybinding specific) input. 

