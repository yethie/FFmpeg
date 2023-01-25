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
 8. Implement the "change" command

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
The following images show the difference between unshaped text and shaped text using two different fonts. In the process of shaping the distance between the letters "V" and "A" is changed and some characters are merged into a single glyph ("fi" in the previous example, "ff" in the second example).

> Written with [StackEdit](https://stackedit.io/).