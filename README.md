# Notes on Joey's fork of the Adafruit GFX Library

This fork of the Adafruit GFX Library aims to support the seamless display of text in all the languages of the world. It achieves this by replacing the standard 5x7 font with the [GNU Unifont](http://unifoundry.com/unifont/index.html), an 8x16 (in some cases 16x16) pixel font that includes glyphs for every Unicode code point in the basic multilingual plane (BMP). It also removes all support for graphic fonts.

![Two e-ink displays, one OLED, a TFT and a memory LCD displaying a variety of Unicode examples: Hello World in ten languages; a series of icons and symbols; a handful of mathematical formulas; and a chapter of the Tao Te Ching in Chinese.](/example.jpg?raw=true)

This should function as a drop-in replacement for the Adafruit GFX Library, as long as you're not using graphic fonts. You can display a Unicode code point by calling `display.writeCodepoint(c)`, where c represents the Unicode code point (not its UTF-8 or UTF-16 representation). You can display a UTF-8 encoded string with the `display.printUTF8(s)` method; I've included a [Very Strict UTF-8 Decoder](https://github.com/douglascrockford/JSON-c/blob/master/utf8_decode.c) that will turn well-formed UTF-8 into code points suitable for display with `writeCodepoint`.

The BMP covers code points from U+0000 to U+FFFF. As configured in this repository, it works with all code points in Plane 0, from U+0020 to U+00FF. For code points outside this range, you can go one of three routes:

* On a **Feather M0 Express**, you can use the included 2-megabyte SPI Flash chip to store the whole Unifont. Useful if you want to support all the languages of the world, such as in an IOT project that draws text from the internet.
* On other boards, you can use the `unifontconvert/converter.py` tool to select just the blocks you need for your project, and store them in program memory. Useful if you need a subset of the BMP, like ASCII and box drawing characters for a menu system, or Latin-1 and Greek for a dictionary of translations.
* You can also combine these two approaches to include the most-needed blocks in program memory for performant display, while falling back to the Flash chip for other codepoints. Useful if, say, you expect to be displaying mostly Cyrillic text, but also want to have occasional access to other scripts, arrows, dingbats or symbols.

The library seamlessly handles both 8x16 and 16x16 glyphs, as well as non-spacing glyphs like accents. Easily display text from any language with a standard encoding — no need to switch code pages or add custom bitmaps just because you dared to want a degree symbol and an umlaut at the same time.

## Storing Unifont on SPI Flash

To take advantage of the whole basic multilingual plane on a Feather Express:

1. Copy the included `unifontconvert/unifont.bin` file to the CircuitPython file system that comes with your Feather M0 Express or Feather M4 Express. It should appear as a volume named `CIRCUITPY` when you plug in your board.
2. When you write your sketch, initialize your display device as usual, but after calling `display.begin()`, call `display.loadUnifontFile()`.

If you have already loaded an Arduino sketch onto your board, the `CIRCUITPY` volume will not appear. You can follow these steps to install `unifont.bin`:

1. While plugged in to your development machine, double-click the RESET button on your board. You should see a volume called `FEATHERBOOT` appear.
2. If you wish to preserve your existing sketch, copy the `CURRENT.UF2` from `FEATHERBOOT` to your hard drive.
3. Download the [latest CircuitPython release](https://github.com/adafruit/circuitpython/releases/tag/3.1.2) for your Feather — it should be a .UF2 file — to the root of the `FEATHERBOOT` volume. This will erase any existing sketch on the board.
4. The `CIRCUITPY` volume should appear. Copy `unifont.bin` to the `CIRCUITPY` volume.

At this point, if you wish to restore your previous sketch, repeat step 1 and drag the backed-up `CURRENT.UF2` to the `FEATHERBOOT` volume; it will reboot to your sketch. Alternately, you can write a new sketch in the Arduino IDE and build normally; even though you will never see the `CIRCUITPY` volume again, `unifont.bin` will remain safely on the SPI flash chip unless you erase it.

Note that seeking through the font file on Flash is somewhat slower than reading glyphs from program memory. If your project primarily uses characters from one Unicode block, you might still want to follow the steps in "Using a subset of Unifont" to store that block in progmem.

## Using a subset of Unifont

If your board lacks an SPI flash chip (or you're using it for something else), you can also select just the subset of the BMP that you expect to need, and store it in program memory. Some blocks you might need: block 20 (symbols, superscript and subscript numbers); block 21 (fractions, arrows); block 25 (box drawing characters); blocks 26-27 (symbols and dingbats).

To select the blocks you want for your project:

1. In a terminal window, navigate to the `unifontconvert` directory.
2. Run `python3 converter.py`, and select option 2 from the main menu.
3. Select the blocks you wish to include in the Unifont font definition file, then enter `done` when finished.
4. Select option 3 from the main menu to generate `glcdfont.c`. 
5. Replace the `glcdfont.c` in the library root with the file generated by the converter utility.

If you are not using the unifont.bin file at all, you will also want to remove `#define UNIFONT_USE_SPI_FLASH` from line 11 of `Adafruit_GFX.h`; this will save about 10 kilobytes of program storage.

## Errata

There are some edge cases that I hope to address eventually. These are the edge cases, and the current workarounds (which I fully intend to break once I fix the underlying behavior).

* Combining marks in canonically ordered strings will appear off by one (i.e. `José` is `U+004A U+006F U+0073 U+0065 U+0301`, but this library will display that as `Jose´`). To coax it into displaying the string correctly, you need to include all combining marks before the character they modify (i.e. `U+004A U+006F U+0073 U+0301 U+0065`). **I hope to fix this behavior in a future update.**
* You currently need to reverse the order of right-to-left scripts like Arabic and Hebrew to get them to display correctly. **In the future I hope to add an RTL mode to the display library.**
* Arabic appears as isolated letterforms instead of connected script, unless you use a [tool like this](https://github.com/artem-azarov/Arabic-Converter-From-and-To-Arabic-Presentation-Forms-B) to convert it to Arabic Presentation Forms. It might make sense to implement this in the display library.

# Original README: Adafruit GFX Library # [![Build Status](https://travis-ci.com/adafruit/Adafruit-GFX-Library.svg?branch=master)](https://travis-ci.org/adafruit/Adafruit-GFX-Library)

This is the core graphics library for all our displays, providing a common set of graphics primitives (points, lines, circles, etc.). It needs to be paired with a hardware-specific library for each display device we carry (to handle the lower-level functions).

Adafruit invests time and resources providing this open source code, please support Adafruit and open-source hardware by purchasing products from Adafruit!

Written by Limor Fried/Ladyada for Adafruit Industries.
BSD license, check license.txt for more information.
All text above must be included in any redistribution.

Recent Arduino IDE releases include the Library Manager for easy installation. Otherwise, to download, click the DOWNLOAD ZIP button, uncompress and rename the uncompressed folder Adafruit_GFX. Confirm that the Adafruit_GFX folder contains Adafruit_GFX.cpp and Adafruit_GFX.h. Place the Adafruit_GFX library folder your ArduinoSketchFolder/Libraries/ folder. You may need to create the Libraries subfolder if its your first library. Restart the IDE.

# Useful Resources

- Image2Code: This is a handy Java GUI utility to convert a BMP file into the array code necessary to display the image with the drawBitmap function. Check out the code at ehubin's GitHub repository: https://github.com/ehubin/Adafruit-GFX-Library/tree/master/Img2Code

- drawXBitmap function: You can use the GIMP photo editor to save a .xbm file and use the array saved in the file to draw a bitmap with the drawXBitmap function. See the pull request here for more details: https://github.com/adafruit/Adafruit-GFX-Library/pull/31

- 'Fonts' folder contains bitmap fonts for use with recent (1.1 and later) Adafruit_GFX. To use a font in your Arduino sketch, \#include the corresponding .h file and pass address of GFXfont struct to setFont(). Pass NULL to revert to 'classic' fixed-space bitmap font.

- 'fontconvert' folder contains a command-line tool for converting TTF fonts to Adafruit_GFX header format.

---

### Roadmap

The PRIME DIRECTIVE is to maintain backward compatibility with existing Arduino sketches -- many are hosted elsewhere and don't track changes here, some are in print and can never be changed! This "little" library has grown organically over time and sometimes we paint ourselves into a design corner and just have to live with it or add ungainly workarounds.

Highly unlikely to merge any changes for additional or incompatible font formats (see Prime Directive above). There are already two formats and the code is quite bloaty there as it is (this also creates liabilities for tools and documentation). If you *must* have a more sophisticated font format, consider creating a fork with the features required for your project. For similar reasons, also unlikely to add any more bitmap formats, it's getting messy.

Please don't reformat code for the sake of reformatting code. The resulting large "visual diff" makes it impossible to untangle actual bug fixes from merely rearranged lines.
