# Log Overlay
 
osu! framework provides in-picture debugging overlays, defined in [`osu.Framework.Input.FrameworkActionContainer`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/FrameworkActionContainer.cs). One of these overlays is the Log Overlay.

## Overview

**Control-F10** toggles the Log Overlay, which displays events logged by [`osu.Framework.Logging.Logger`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Logging/Logger.cs). Invoking [`Logger.Log`](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Logging/Logger.cs#L148) will log messages that are able to be displayed in this stream. 

By default, messages are logged using the logger level [`Verbose`](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Logging/Logger.cs#L479), which get omitted from the log overlay unless building with the `DEBUG` preprocessor flag. Any level higher than verbose will result in the messages being displayed through this overlay regardless.





