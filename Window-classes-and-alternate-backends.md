The framework is in a state of flux, potentially moving away from osuTK (OpenTK) to a different library for providing window / context initialisation. This document aims to explain the classes which are currently present, as there are two different sets of confusingly named classes which are for different backends.


**Everything with GameWindow is osuTK, everything with Window is the new windowing.**

osuTK classes include:
- `GameWindow`
- `DesktopGameWindow`
- `WindowsGameWindow`
- `MacOSGameWindow`
- `LinuxGameWindow`


The new windowing classes take in the backend, of which `Sdl2WindowBackend` provides the SDL-specific implementation. The following classes are using a newer structure (not enabled by default, yet).

new windowing classes include:
- `Window`
- `DesktopWindow`

The `IWindow` interface acts as an interface to allow both of the above to function as a backend for now.

The `IGameWindow` interface is used to store osuTK's game window to delegate our own methods to it.