# bad-apple
This is a QGIS version of the iconic ["Bad Apple" video](https://www.youtube.com/watch?v=FtutLA63Cp8).  This video has been recreated using a wide variety of hardware and software and analog methods.  But, as far as I know, there has not been a version made with QGIS ... until now.

A full-resolution version of the video is available at <https://youtu.be/grtfdMUXxgU>

To run the video in QGIS on your own computer:
1. Download the repo ([link](https://github.com/kgjenkins/bad-apple/archive/refs/heads/main.zip))
2. Open "bad-apple.qgz" in QGIS
3. Open the Temporal Controller panel (View menu > Panels)
4. Click the "play" button


## How does this work?

The original video was captured and saved as individual frames using [ScreenToGif](https://github.com/NickeManarin/ScreenToGif).  I then converted those frames to 1-bit black/white pixels using [IrfanView](https://www.irfanview.com/).  Then I converted each frame into vector multi-polygons (one black, one white) using [gdal_polygonize.py](https://gdal.org/programs/gdal_polygonize.html)

From there, I used QGIS to merge all the polygon layers (one for each frame) into a single geopackage.

One of my goals was to be able to run the video in real time directly within QGIS.  Although it is possible to set up the temporal controller to create a timestamp on-the-fly from the frame number, it is much faster to create a dedicated attribute field containing the timestamp, so I added a `dt` field using the field calculator with the expression `datetime_from_epoch("frame" * 1000)` -- epochs are in milliseconds, so this creates a timestamp where each frame is one second.  (The particular start time isn't important here.)

After trying several different methods for creating various styles, I found that the most efficient way to view a sequence of different styles directly in QGIS seems to be using a single layer with a rule-based style.  Each rule looks something like this:

```
"frame" >=143 AND "frame" < 315 AND "value" = 0
```

which sets up a style for the black pixels of frames 143 to 314.  I experimented with some of the many styles available in QGIS.  The hex and pixelated square grids were imitated using a point pattern fill with the point shape and size corresponding to the spacing.  Smooth transitions between sizes and colors were implemented using data-defined overrides.  I'm not going to explain every style, but feel free to open the Layer Styling Panel to explore how the various styles were defined.  Whenever you see a yellow expression symbol next to a parameter, that indicates that a data-defined override is being used -- click it and edit to view the expression.  In other cases, the assistant was used instead of a hand-written expression.

I have the temporal controller set to use a frame rate of 30 frames per second, which was the frame rate captured from the original video.  On my computer, QGIS does a fairly smooth rendering, although it is not quite 30 frames per second.  QGIS skips frames as necessary to keep up the speed of the video motion.  I had to omit advanced draw effects like "drop shadow" and "outer glow" to keep the actual frame rate from dropping too low, although it would have been fine if I just wanted to export the frames, when it can take as long as neededd to fully render each frame.

After exporting all the frames, I used ScreenToGif to combine them into a .mp4 video at 34ms per frame.  (ScreenToGif doesn't allow floating point values for the frame duration, so I couldn't set it to 33.333333 -- maybe I should have done the capture at 40fps, which would be an even 25ms.)

I then used ShotCut to splice together the view in QGIS with the exported full-screen version, and add the audio from the original video.  Due to imprecise frame rates during the original frame capture, I had to make some minor adjustments to keep the audio and video aligned.  It's not perfect, but pretty close.
