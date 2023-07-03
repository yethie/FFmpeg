# Enhanced ffmpeg drawtext video filter
## Introduction
The main reason for me to modify the drawtext filter, is that I needed a way to write text on a video that is consistent with other tools, like text editors, browsers, or GIMP. I mean: what users see on the other tools should be almost identical to what they see on the video. The changes I implemented to achieve this objective (and some other minor features I needed in my project, but that may be useful for all users) are:

 1. Use the libharfbuzz library to shape text
 2. Use the font-defined line height as default spacing between lines
 3. Introduce 1/4 pixel precision for glyph positioning
 4. Extend the concept of "box" around text, allowing the user to specify its with and height or to have them be computed automatically
 5. Allow users to specify text alignment in horizontal and vertical directions
 6. Allow users to specify what the "y" parameter is referred to
 7. Add some variables that the users can insert in expressions (to add more flexibility to text positioning)
 8. Support a subset of the filter options as commands

## Build
The filter depends on the libharfbuzz shaping library, which must be enabled using the new "--enable-libharfbuzz" configuration option. Without this option the filter is not compiled at all.

## Filter options

### New options
`boxw, boxh` force the width and height of the box around text

`text_align` set the text vertical and horizontal alignment

`y_align`define what the `y` parameter should be referred to

### Modified options
The syntax of `boxborderw` was extended to allow each border size to be specified independently:

 - `boxborderw=10` set the width of all the borders to 10
 - `boxborderw=10|20` set the width of the top and bottom borders to 10 and the width of the left and right borders to 20
-  `boxborderw=10|20|30` set the width of the top border to 10, the width of the bottom border to 30 and the width of the left and right borders to 20
-  `boxborderw=10|20|30|40`  set the borders width to 10 (top), 20 (right), 30 (bottom), 40 (left)

## Changed behavior
The height of a line is now equal to the one defined in the font metrics. It was equal to the height of highest glyph in the text, thus being dependent on the text itself.

## New variables
 - `font_a` the ascent size defined in the font metrics
 - `font_d` the descent size defined in the font metrics
 - `top_a` the maximum ascender of the glyphs of the first text line
 - `bottom_d` the maximum descender of the glyphs of the last text line

## Examples
### Text shaping
The following images show the difference between unshaped text and shaped text using two different fonts. In the process of shaping the distance between the letters "V" and "A" is changed and some characters are merged into a single glyph ("ff" in the first example, "fi" in the second example). For each font the first image was rendered using the current filter implementation, the second one was rendered using the modified filter.

font: *DejaVu Serif*

![unshaped text](https://user-images.githubusercontent.com/27201066/214627926-c3b5c6a4-ba83-4e4b-82b8-7bdc8790770c.png)

![shaped text](https://user-images.githubusercontent.com/27201066/214627474-a8cc9fa8-c3a3-493e-95fe-86363aca0710.png)

font: *PT Serif Regular*

![unshaped text](https://user-images.githubusercontent.com/27201066/214628371-fc71c23e-edd3-459e-8ac9-73a69ae0ca02.png)

![shaped text](https://user-images.githubusercontent.com/27201066/214628436-89b87c62-d607-4ba6-b752-6ba6de16d49e.png)

### Box size and text alignment

![box size and text align](https://user-images.githubusercontent.com/27201066/214628965-82e3303b-c378-4c13-906b-9012b4174faa.png)

### Y alignment
In the following image the blue line was placed at the y value specified as an option to the drawtext filter. The two words are drawn using two instances of the drawtext filter with the same `y` value. The effect of the three possible values of the y_align option are shown. The value `text` mimics the behavior of the current filter implementation.

![y_align](https://user-images.githubusercontent.com/27201066/214629259-a57dce1c-c112-47cb-a3c5-8625e51e5102.png)

### Default line height
When writing multiline text the current filter implementation uses a line height value that depends on the glyphs contained in the text. Using the line height defined in the font metrics gives a better looking and more stable result. In the following images two instances of drawtext filter are used to draw two texts with three lines each.

*Current filter implementation*: the line height depends on the text

![line_height-old](https://user-images.githubusercontent.com/27201066/214710175-536e5f4c-3711-4abf-b133-0b62e875fb7d.png)

*Modified filter*: the line height does not depend on the text

![line_height-new](https://user-images.githubusercontent.com/27201066/214710200-453d3c4d-fe9a-4e97-a68e-887d127f55f6.png)

### Subpixel precision
Subpixel precision enables smoother text animations. In the first video (rendered using the current filter implementation) the text movement along the "y" axis is a bit jerky, while it is smoother in the second video (rendered with the modified filter).

https://user-images.githubusercontent.com/27201066/214630731-7dc91908-59ec-495c-be1c-a51eb50ba607.mp4

https://user-images.githubusercontent.com/27201066/214632295-3aa439ef-0835-40cd-8e9f-3ef483af4b7d.mp4

### Commands
The "reinit" command causes the re-initialization of the whole filter configuration, the font file is reloaded and glyphs are re-rendered. This causes an overhead that can be avoided when the command is not used to change the font. In the following video the "x", "y" and "text" commands were used to modify the text and its position at each frame, the rendering was about 10 times faster than using the "reinit" command.

https://user-images.githubusercontent.com/27201066/214634079-21ba9418-c9bf-4b33-aac3-98670c3fb67c.mp4
