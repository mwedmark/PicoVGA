Guide how to create and insert pictures into the vga_monoscope project.
NOTE: I'm not the creator of this project but only tried to learn about Pi Pico with it. I'm using mostly Windows so this description is made using Windows.

Tools and versions used:
Paint.NET 4.3.11
GIMP 2.10.32 (rev.1)

First you need to be comfortable with compiling these examples using the ARM toolchain and the tools included here in the Repo. Then you probably want to use a serial client (I've used Putty) to be able to control which resolution that is currently used in your Pico. 115200 baud is used and char's (0-9, A-V) is sent to change resolution.

RESTRICTIONS
There are quite extensive restrictions when trying to convert your average high-res multi-color JPG to the formats supported by PicoVGA. 

RESTRICTIONS-COLORS:
PicoVGA Colorspace covers 256 colors using 3-bit R, 3-bit G and 2-bit B. Because of the non-uniform bit length the best greyscale colors only uses 2-bit colors, thus only 4 colors. This is less than, for example, C64 which actually has 5 colors from black to white. But by using grey colors slighty off-red/off-green/off-yellow you can extend this to at least 8 colors. Another possible thing to do is to instead use Black-Red-White, Black-Green-White, Black-Red-Yellow-White kind of gradients and map these onto your greyscale pictures to extend the dynamics of B/W pictures. I think that this also gives a rather nice artistic touch to your pictures.

RESTRICTIONS-DITHER-SIZE:
PicoVGA try to limit memory usage and simplfy memory structure using two main restrictions: 8-bit color per pixel and simple custom RLE-compression which is unpacked in real-time using PIO's.

Here's an example: 
A full 640x480x8-bit color pixture takes => 307'200 bytes unpacked. A maximum size that works for vga_monoscape examples seem to be around 170KB-200KB although the tools state 256K being the maximum. So to be able to use a picture in any resolution it need to be able to be packed from its original size down to 170KB. The limitation comes from 264KB of RAM memory in Pico and the algorithm for handling the pictures are also using part of that. In the case of standard VGA 640x480@8-bit that pictures need to be packed into half its original size. Because of the nature of the simple RLE-encoding algorithm, the worst case picture is a fully dithed one. So for a picture to actually fit, the smallest possible amount of dithering possible must be used. In the above example, a good measurement is that no more than half the picture content can be dithered. So by setting a lower dither amount in Paint.NET when doing Quantizing, the target picture size will be smaller. Here you will need to try different settings until you are satiesfied. The lower the resolutions, higher possible dithering settings can be used. And higher resolutions, the harder it will get to actually fit a photo picture into the limiting memory at all.

Paint.NET:
The reason for me choosing to use a combination of Paint.NET and GIMP is that they, as far as I've seen, have different ways of handling Palettes and Dithering. Paint.NET can only handle palette with maximum 96 colors so it is not possible to use the whole PicoVGA palette, even with plug-ins. I've tried the following plug-ins: "Recolor Using Palette" or "TR's Custom Palette Matcher" but both are limited to Paint.NET maximum 96 colors.
But the built-in filter "Quantize" (found under Effecets/Color/Quantize) does a rather good job of limiting bitdepth per color and a t the same time being albe to apply custom Dithering. So we use Paint.NET to limit color depth and apply dithering.

GIMP:
GIMP is better at handling arbitrarily length palettes so it is possible to import the full PicoVGA palette. This is how to import the palette file into GIMP:
- 

- Choose a file that you want to show on the PiPico using PicoVGA.
- Scale the picture to a suitable resolution (one of the resolutions available). On "normal" flat-screen TFT computer displays none of the PAL/NTSC resolutions seem to work, so if you are new I recommend trying out one of these: letters G=>V in Putty (320x200 => 1360x768).
