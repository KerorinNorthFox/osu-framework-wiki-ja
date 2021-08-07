# Framework Key Bindings
**osu!framework** has framework key bindings in [`FrameworkActionContainer`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/FrameworkActionContainer.cs) for specific actions listed below:
| Key binding | Framework action |
|:-----------:|:-------------------- |
| <kbd>Ctrl</kbd>+<kbd>F1</kbd> | Toggles the [draw visualiser](/ppy/osu-framework/wiki/Debug-Overlays:-Draw-Visualizer). |
| <kbd>Ctrl</kbd>+<kbd>F2</kbd> | Toggles the global statistics (and enabled performance logging while visible). |
| <kbd>Ctrl</kbd>+<kbd>F3</kbd> | Toggles the texture atlas viewer. |
| <kbd>Ctrl</kbd>+<kbd>F7</kbd> | Cycle frame limiter modes. |
| <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F7</kbd> | Cycle threading execution modes (single-/multithreaded). |
| <kbd>Ctrl</kbd>+<kbd>F9</kbd> | Toggles the audio mixer viewer. |
| <kbd>Ctrl</kbd>+<kbd>F10</kbd> | Toggles the [log overlay](/ppy/osu-framework/wiki/Debug-Overlays:-Log-Overlay). |
| <kbd>Ctrl</kbd>+<kbd>F11</kbd> | Cycles through the [frame statistics](/ppy/osu-framework/wiki/Debug-Overlays:-Frame-Statistics-Overlay) view states. (`Minimized`, `Extended`) |
| <kbd>Alt</kbd>+<kbd>Enter</kbd> / <kbd>F11</kbd> | Cycles through the game window view modes. (`Windowed`, `Fullscreen`, `Borderless`) |

In addition, `TestBrowser` implementation also allow for the following key bindings:

| Key binding | Action |
|:-----------:|:-------------------- |
| <kbd>Ctrl</kbd>+<kbd>H</kbd> | Hide the tests list sidebar |
| <kbd>Ctrl</kbd>+<kbd>F</kbd> / <kbd>Cmd</kbd>+<kbd>F</kbd> | Focus the search box |
| <kbd>Ctrl</kbd>+<kbd>R</kbd> / <kbd>Cmd</kbd>+<kbd>R</kbd>| Reload the currently displayed test scene |