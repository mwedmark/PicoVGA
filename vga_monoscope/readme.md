# Guide to create and integrate new pictures into the vga_monoscape example project for Pi Pico
Guide how to create and insert pictures into the vga_monoscope project.
NOTE: I'm not the creator of this project but only tried to learn about Pi Pico with it. I'm using mostly Windows so this description is made using Windows.

Tools and versions used:
- Paint.NET 4.3.11
- GIMP 2.10.32 (rev.1)

First you need to be comfortable with compiling these examples using the ARM toolchain and the tools included here in the Repo. Then you probably want to use a serial client (I've used Putty) to be able to control which resolution that is currently used in your Pico. 115200 baud is used and char's (0-9, A-V) is sent to change resolution.

## RESTRICTIONS
There are quite extensive restrictions when trying to convert your average high-res multi-color JPG to the formats supported by PicoVGA. 

### RESTRICTIONS-COLORS:
PicoVGA Colorspace covers 256 colors using 3-bit R, 3-bit G and 2-bit B. Because of the non-uniform bit length the best greyscale colors only uses 2-bit colors, thus only 4 colors. This is less than, for example, C64 which actually has 5 colors from black to white. But by using grey colors slighty off-red/off-green/off-yellow you can extend this to at least 8 colors. Another possible thing to do is to instead use Black-Red-White, Black-Green-White, Black-Red-Yellow-White kind of gradients and map these onto your greyscale pictures to extend the dynamics of B/W pictures. I think that this also gives a rather nice artistic touch to your pictures.

### RESTRICTIONS-DITHER-SIZE:
PicoVGA try to limit memory usage and simplfy memory structure using two main restrictions: 8-bit color per pixel and simple custom RLE-compression which is unpacked in real-time using PIO's.

Here's an example: 
A full 640x480x8-bit color pixture takes => 307'200 bytes unpacked. A maximum size that works for vga_monoscape examples seem to be around 150KB-180KB although the tools state 256K being the maximum. So to be able to use a picture in any resolution it need to be able to be packed from its original size down to 170KB. The limitation comes from 264KB of RAM memory in Pico and the algorithm for handling the pictures are also using part of that. In the case of standard VGA 640x480@8-bit that pictures need to be packed into half its original size. Because of the nature of the simple RLE-encoding algorithm, the worst case picture is a fully dithed one. So for a picture to actually fit, the smallest possible amount of dithering possible must be used. In the above example, a good measurement is that no more than half the picture content can be dithered. So by setting a lower dither amount in Paint.NET when doing Quantizing, the target picture size will be smaller. Here you will need to try different settings until you are satiesfied. The lower the resolutions, higher possible dithering settings can be used. And higher resolutions, the harder it will get to actually fit a photo picture into the limiting memory at all.

## Paint.NET:
The reason for me choosing to use a combination of Paint.NET and GIMP is that they, as far as I've seen, have different ways of handling Palettes and Dithering. Paint.NET can only handle palette with maximum 96 colors so it is not possible to use the whole PicoVGA palette, even with plug-ins. I've tried the following plug-ins: "Recolor Using Palette" and "TR's Custom Palette Matcher" but both these are limited to the Paint.NET's limitation of 96 color palettes.

The built-in filter "Quantize" (found under Effecets/Color/Quantize) does a rather good job of limiting bit depth per color and at the same time being able to apply custom dithering. So we use Paint.NET to limit color depth and apply dithering.

## GIMP:
GIMP is better at handling arbitrarily length palettes so this makes it possible to import the full PicoVGA palette. This is how to import the palette file into GIMP:
- Find the file to import in the original structure of PicoVGA under: PicoVGA/_picovga/_exe/pal332.act
    - Find the palette tab window on the right in GIMP. 
    - Right-click on the existing palettes and choose "Import Palette".
    - Choose Source: Palette File and press field to the right that to choose file via file dialog.
    - Select pal332.act and select open.
    - Make sure to select pal332.act as the active Palette!
Now you have prepared GIMP to be able to understand the limitations of PicoVGA!

## HOW TO - STEP BY STEP DESCRIPTION:
- Choose a file that you want to show on the PiPico using PicoVGA.
- Open image in Paint.NET
- Scale the picture to a suitable resolution (one of the resolutions available). On "normal" flat-screen TFT computer displays none of the PAL/NTSC resolutions seem to work, so if you are new I recommend trying out one of these: letters G=>V in Putty (320x200 => 1360x768). A good start is the normal 640x480!
- If you're going for grey-scale pictures: apply Adjustments/Black and White
- Perform Quantize by choosing Effects/Color/Quantize
    - Choose Algorithm for dithering: A prefer Octree
    - Choose number of colors: Go for around 8-40 colors. You need to try this out
    - Choose dithering level: 0-8 either wich looks better or the level you think will fit into memory (see above)
    - Transparant level: 0 (not sure what this does at all)
- Save As: BMP with settings Bit depth: 8-bit color, dithering level:0 (already applied)
- Make sure that the setting: Image/Mode is set to RGB to start with
- Now open this BMP file in GIMP.
- Make sure to have the new imported pal332.act Palette file chosen
- Choose Image/Mode Indexed..

- Choose Export Image.. and select NameOfImage.BMP and use the settings:
    - Run-Length Enconding - Deselect
    - Compatibility Options - DO not write Color space info - Deselect

- Press Export button
- Now goto folder: ..\PicoVGA\vga_monoscope\img\ in Explorer.
- Make sure to rename the original BMP-file with the correct resolution to suffix _OLD. For example, in the above image we had 640x480 so rename: monoscope_640x480.bmp => monoscope_640x480_OLD.bmp.
- Then copy your created file: 'NameOfImage.BMP' into the \img\ and rename that to: 
'monoscope_640x480.bmp'.
- Now execute the script called: 'convert.bat'. That will convert all BMP-files to C/C++ source files with data. Now your file will be used in the tool-chain next time you build the monoscope example!
- Last step needed to be done is to specify the correct binary size of the picture, so it is the same size in: '..\PicoVGA\vga_monoscope\img\monoscope_640x480.cpp' and '..\PicoVGA\vga_monoscope\src\main.h'.  

### ** NOTE **
If you try to build the project before fixing this there will be a clear error message shown: 

>img/monoscope_640x480.cpp:74:10: error: conflicting declaration 'const u8 Monoscope640x480 [175304]'
>   74 | const u8 Monoscope640x480[175304] __attribute__ ((aligned(4))) = {
>      |          ^~~~~~~~~~~~~~~~
>In file included from ./src/include.h:16,
>                 from img/monoscope_640x480.cpp:1:
>./src/main.h:53:17: note: previous declaration as 'const u8 Monoscope640x480 [152484]'
>   53 | extern const u8 Monoscope640x480 [152484] __attribute__ ((aligned(4)));
>      |                 ^~~~~~~~~~~~~~~~
>make: *** [../Makefile.inc:458: build/monoscope_640x480.o] Error 1

- Search the following line in 'monoscope_640x480.cpp' that looks like this:
const u8 Monoscope640x480[175304] __attribute__ ((aligned(4)))
Then find the similar line in 'main.h'
extern const u8 Monoscope640x480 [152484] __attribute__ ((aligned(4)));

- Copy the value from 'monoscope_640x480.cpp' (here 175304) and overwrite value in 'main.h' (here 152484) so they both show the same number, end result should be:
'main.h'
extern const u8 Monoscope640x480 [175304] __attribute__ ((aligned(4)));

- Save the updated 'main.h'

- Goto folder ..\PicoVGA\vga_monoscope and execute build by typing C and Enter! Make sure eveything works and a new program.uf2 file is created.
- Reboot your Pico and copy the file: PicoVGA\vga_monoscope\program.uf2 onto your Pico via Windows Explorer
- The Pico will reboot. Just make sure to reconnect Putty via the serial port to Pico and choose the correct resolution that you've changed! You should now see your new cusotm picture betgin shown on-screen! Congrats!
