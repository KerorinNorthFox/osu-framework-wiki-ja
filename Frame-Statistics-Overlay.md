# Frame Statistics

osu! framework provides in-picture debugging overlays, defined in `osu.Framework.Input.FrameworkActionContainer`. One such overlay is the Frame Statistics Overlay.

## Overview
**Control-F11** toggles the frame statistics overlay. The first time you toggle this overlay, you will be greeted with a minimized version of it. This view displays the frame timings of the four main gameplay threads: Audio, Input, Update, and Draw. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538264876561203200/Screen_Shot_2019-01-25_at_3.31.08_PM.png)

**Pressing Control-F11 again** will expand the overlay, displaying a graph of the activity on each thread. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538266387014221824/Screen_Shot_2019-01-25_at_4.58.06_PM.png)

**Holding down the control key** in this state will expand the graph further, allowing you to peek at how many events of each type are occuring on each thread. Statistics are defined in .

![](https://cdn.discordapp.com/attachments/318886668889227266/538268818951241754/unknown-2_copy.png)

## Statistics

The statistics for this overlay are defined in `osu.Framework.Statistics.FrameStatistics`. 

* Invalidations: The amount of Invalidate() calls performed on cached objects, defined in `osu.Framework.Caching.Cached`.
* Refreshes: The amount of Validate() calls performed on cached objects, defined in `osu.Framework.Caching.Cached`.
* DrawNodeCtor: The amount of new `osu.Framework.Graphics.DrawNode`s created.
* DrawNodeAppl: The amount of `osu.Framework.Graphics.DrawNode`s updated due to a disagreement between the Drawable's Invalidation ID and the node's Invalidation ID. See `osu.Framework.Graphics.Drawable`.
* ScheduleInvk: The amount of scheduled tasks not yet performed 

