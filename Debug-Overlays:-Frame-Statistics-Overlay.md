# Frame Statistics

osu! framework provides in-picture debugging overlays, defined in [`osu.Framework.Input.FrameworkActionContainer`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/FrameworkActionContainer.cs). One such overlay is the Frame Statistics Overlay.

* [Overview](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#overview)
  * [Minimized View](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#minimized-view)
  * [Extended View](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#extended-view)
  * [Detailed View](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#detailed-view)
* [Statistics](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#statistics)
  * [Update](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#update)
  * [Draw](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#draw)
  * [Audio](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#audio)
  * [Input](https://github.com/ppy/osu-framework/wiki/Frame-Statistics-Overlay#input)

## Overview

### Minimized View
**Control-F11** toggles the frame statistics overlay. The first time you toggle this overlay, you will be greeted with a minimized version of it. This view displays the frame timings of the four main gameplay threads: Audio, Input, Update, and Draw. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538264876561203200/Screen_Shot_2019-01-25_at_3.31.08_PM.png)

### Extended View
**Pressing Control-F11 again** will expand the overlay, displaying a graph of the activity on each thread. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538266387014221824/Screen_Shot_2019-01-25_at_4.58.06_PM.png)

### Detailed View
**Holding down the control key** in this state will expand the graph further, allowing you to peek at how many events of each type are occuring on each thread.

![](https://cdn.discordapp.com/attachments/318886668889227266/538268818951241754/unknown-2_copy.png)

## Statistics

The statistics for this overlay are defined in [`osu.Framework.Statistics.FrameStatistics`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Statistics/FrameStatistics.cs). 

These statistics are counted per frame.

### Update

* [Invalidations](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Caching/Cached.cs#L73): The number of Invalidate() calls performed on [cached](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Caching/Cached.cs) objects.
* [Refreshes](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Caching/Cached.cs#L83): The number of Validate() calls performed on [cached](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Caching/Cached.cs) objects.
* [DrawNodeCtor](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Drawable.cs#L1691): The number of new [DrawNodes](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/DrawNode.cs) created.
* [DrawNodeAppl](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Drawable.cs#L1697): The number of [DrawNodes](https://github.com/ppy/osu-framework/blob/naster/osu.Framework/Graphics/DrawNode.cs) updated due to a disagreement between the [Drawable's](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/Drawable.cs) Invalidation ID and the node's Invalidation ID.
* [ScheduleInvk](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Containers/CompositeDrawable.cs#L721): The number of [timed tasks](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Threading/Scheduler.cs) ran from the queue.
* [InputQueue](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/InputManager.cs#L288): The number of [UserInputManagers](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/UserInputManager.cs) handling non-positional (keybinding specific) input. 
* [PositionalIQ](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/InputManager.cs#L315): The number of [UserInputManagers](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/UserInputManager.cs) handling positional input.

### Draw
* [VBufBinds](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/OpenGL/GLWrapper.cs#L169): The number of [OpenGL](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Graphics/OpenGL/GLWrapper.cs) [VBO](https://www.khronos.org/opengl/wiki/Vertex_Specification#Vertex_Buffer_Object) bind calls.
* [VBufOverflow](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Batches/VertexBatch.cs#L107): The number of times the vertex count has exceeded the number allocated for the current VBO.
* [TextureBinds](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/OpenGL/GLWrapper.cs#L218): The number of times [glBindTexture](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glBindTexture.xhtml) has been called. 
* [DrawCalls](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Batches/VertexBatch.cs#L139): The number of active draw calls.
* [ShaderBinds](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/OpenGL/GLWrapper.cs#L620): The number of times [glUseProgram](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glUseProgram.xhtml) has been called.
* [VerticesDraw](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/Batches/VertexBatch.cs#L140): The number of vertices drawn using [glDrawElements](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glDrawElements.xhtml).
* [VerticesUpl](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/OpenGL/Buffers/VertexBuffer.cs#L121): The number of vertices updated using [glBufferSubData](https://www.khronos.org/registry/OpenGL-Refpages/es2.0/xhtml/glBufferSubData.xml).
* [Pixels](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Graphics/OpenGL/Textures/TextureGLSingle.cs#L265): The amount of area covered by drawn elements.

### Audio

* [TasksRun](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Audio/AudioComponent.cs#L63): The number of actions in queue.
* [Tracks](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Audio/Track/Track.cs#L126): The number of [tracks](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Audio/Track/Track.cs) being updated.
* [Samples](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Audio/Sample/SampleManager.cs#L64): The number of [samples](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Audio/Sample/Sample.cs) being updated.
* [SChannels](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Audio/Sample/SampleChannel.cs#L46): The number of [SampleChannels](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Audio/Sample/SampleChannel.cs) being updated.
* [Components](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Audio/AudioComponent.cs#L64): The number of [AudioComponents](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Audio/AudioComponent.cs) being updated.

### Input

* [MouseEvents](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/Handlers/Mouse/OsuTKMouseHandlerBase.cs#L57): The number of mouse events being handled by [InputHandlers](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/Handlers/InputHandler.cs).
* [KeyEvents](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/Handlers/Keyboard/OsuTKKeyboardHandler.cs#L58): The number of keyboard events being handled by [InputHandlers](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/Handlers/InputHandler.cs).
* [JoystickEvents](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/Handlers/Joystick/OsuTKJoystickHandler.cs#L43): The number of joystick events being handled by [InputHandlers](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Input/Handlers/InputHandler.cs).

