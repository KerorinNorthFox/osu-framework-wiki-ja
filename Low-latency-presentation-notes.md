This page is for the sole purpose of keeping track of R&D efforts towards getting the lowest latency non-tearing frame presentation possible.

## reflex-like implementations:
- [GitHub - ishitatsuyuki/LatencyFleX: Vendor agnostic latency reduction middleware. An alternative to NVIDIA Reflex.](https://github.com/ishitatsuyuki/LatencyFleX)
	- [GitHub - smoogipoo/osu-framework at latencyflex](https://github.com/smoogipoo/osu-framework/tree/latencyflex)
	- [GitHub - smoogipoo/veldrid at latencyflex](https://github.com/smoogipoo/veldrid/tree/latencyflex)
- [vsync_blurbusters/render_vsync_demo.cpp at main · ad8e/vsync_blurbusters · GitHub](https://github.com/ad8e/vsync_blurbusters/blob/main/render_vsync_demo.cpp)
- [CVDisplayLinkOutputCallback | Apple Developer Documentation](https://developer.apple.com/documentation/corevideo/cvdisplaylinkoutputcallback?language=objc)

## CV/CADisplayLink don't work well

- https://thume.ca/2017/12/09/cvdisplaylink-doesnt-link-to-your-display/ (make sure to read edit)
- https://www.gamedeveloper.com/programming/game-loops-on-ios

## General information
[An input lag investigation - General - Libretro Forums](https://forums.libretro.com/t/an-input-lag-investigation/4407/726)
[wait until Vsync API · Issue #1157 · glfw/glfw · GitHub](https://github.com/glfw/glfw/issues/1157)




