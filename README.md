Bare-metal Teensy 3.x Development
(Last modified 18 May 2014)

Updated 19 May 2014
Oops!  Found another problem with my crt0.s file; I had some of the UART vector addresses messed up.  Grab the latest in the updated zip archive below.


Updated 18 May 2014
I've fixed several bugs in the blinky project.  The project itself compiled and ran, but there were errors in somecomponent files that would cause issues if you tried to expand blinky for other uses.  The biggest change is to the crt0.s file.  The version in the original blinky project had a vector table taken from another K20 design.  Unfortunately, that design did not use an MK20DX256 family chip, so for the Teensy, the vector table was missing important vectors or had them assigned to the wrong addresses.  When I tried to expand the blinky project to support UART interrupts, all non-core interrupts ended up causing a hard fault.  Redoing crt0.s with the correct vector table fixed this problem.

I also made some changes to the blinky.mak file.  One of the fixes was to modify the sed script so it correctly reformatted errors for Visual Studio; turns out the key was enclosing the sed script in single-quotes, not double-quotes.  I have a version of this script running on a different machine that correctly handles double-quotes, but another machine doesn't; looks like single-quotes are universal.

I've updated the zip file link below with the newest blinky code.  This zip file is now about 5.8 MB (yes, megs).  The bulk of this is taken up by the entire suite of Freescale Kinetis header files.  I did this because it was so freaking hard to get my hands on the correct MK20DX256 header file.  I ended up pulling the entire set of files from a Code Warrior distribution provided by Freescale; hopefully, this will save someone else the pain I had to go through.


I recently picked up one of the Teensy 3.1 boards.  I'm blown away with the whole design; the tiny footprint, the amount of usable I/O brought out, the experimenter-friendly layout, the on-chip resources, and most of all, the price.  $19 for 256 KB of flash, 64 KB of RAM, all in a Cortex-M4 ARM that can hit 72 MHz.  What's not to like?

Well, I did find one thing I don't care for in the Teensy, which is why this page exists.  You are expected to use Arduino dev tools to write your Teensy apps, and I'm not an Arduino fan.  Yes, Arduino has brought programming to people who otherwise might not have been able to enjoy this hobby.  But Arduino is not for me, for various reasons.  So here is a set of instructions for setting up a bare-metal dev environment for the Teensy 3.x.

These instruction are fairly high level.  You can find more detail on a similar project on my bare-metal mbed page.  Note that this site owes a lot to others who have set up their version of bare-metal dev on the Teensy, or who have contributed to Teensy dev in general.  What you see here is a mashup of various websites and archive files; where possible, I've added notes in the source files detailing names and websites of others whose help I've used..


CodeSourcery
CodeSourcery G++ Lite is a gcc tool suite suitable for developing ARM-based (and other) projects.  You can download the CodeBench Lite (command-line only)  suite from Mentor Graphics.  Here is a link to the download page for the ARM EABI suite, which works with the Teensy 3.x board: https://sourcery.mentor.com/sgpp/lite/arm/portal/release1802.  Since I'm using Windows XP, I chose the IA32 Windows Installer from the list of recommended packages.  Save this installer to a folder on your desktop, then double-click the installer to launch it.

Note that when you install, CodeSourcery will provide a default install folder; DO NOT use the default!  Provide a new install path that does NOT contain spaces in the path!  I used c:\CodeSourcery.  Immediately after the install finishes, use the Windows file explorer to open the install folder and rename the folder \Sourcery G++ Lite to \SourceryG++Lite, again, to eliminate spaces in the path to the various CS tools.  If you don't do these steps, you will have all kinds of issues later when you try to build one of your projects.


make
You will need a make utility for your host OS.  Since I'm using Windows XP, I picked up a Windows-friendly version from the Free Software Foundation; GNU Make 3.81, built for i386-pc-mingw32, available here.


Microsoft's Visual Studio 2008 Express Edition
CodeSourcery provides the compler, linker, assembler, and other common gcc tools, but does not provide an integrated development environment (IDE).  I chose to use Visual Studio 2008 Express Edition (VS2008) for my IDE.

I have used Visual Studio as an IDE for years, both for desktop and embedded development.  I like how VS lets me organize my files and projects, I like the syntax highlighting, the tooltips on hover, the auto-suggest when working with variables and structures, the right-click on a definition to hop to the definition.  I also use the ability to double-click on an error and go right to the offending line so much that I get frustrated when an IDE doesn't provide this.  Yes, there are lots of IDEs out there, and you might find one you like more than VS.  If so, go ahead and enjoy.

Note that this double-click on error feature is not available when compiling with gcc in VS2008.  However, a bit of sed magic in the makefile fixes this lack.  You can find details below.


The Kinetis source file and docs
You will need a set of source files for Freescale's Kinetis K20 microcontroller, the device used in the Teensy 3.x boards.  There are many of these floating around on the web.  They appear to have a common origin, apparently either Code Warriror or Freescale devs.  At one point, I had three different versions of these source files and the final blinky project uses a blend of all of them.  Probably the best starting point is to grab the kinetis_50MHz_sc package, available here in the Lab and Test Software section.  Unzip this archive, then open the k20d50m_sc_baremetal folder.  Many of the files you will be using can be found in this folder.

You will also need the K20 documentation.  The main items are the Reference Guide (K20_QuickRefGuide.pdf) and the Reference Manual (K20P64M72SF1_RefMan.pdf).  These are available on the Freescale K20 page here.


The project layout
I have copied files from the K20 archive listed above, then moved them into folders in my Teensy3x project tree in a layout I like, which is different from the layout assumed in the Freescale projects.  My layout looks like this:

'c:\projects\Teensy3x\
    blinky\
        blinky.sln        (VS2008 solution file for blinky)
        blinky\
            blinky.c
            blinky.mak
    common\
        arm_cm4.c
        crt0.s
        sysinit.c
        Teens31_flash.ld
    support\
        (assorted low-level .c files, such as uart.c, not needed for blinky)
    include\
        arm_cm4.h
        common.h
        MK20D7.h
        sysinit.h
        (assorted low-level .h files, such as uart.h, not needed for blinky)
'
This layout lets me keep all project-related source files in a dedicated project folder, but find common driver code and include files easily.  To duplicate this layout, you will need to find the associated files in the K20 archive and move them as needed.  Or you can create your own layout, then modify the blinky.mak makefile accordingly.


Defining a project in VS2008 with GCC
Create a new project in VS2008, of type General and using the Makefile template.  Provide a name for the project and a path to where you will keep your source files. In my case, the solution file will be in c:\projects\Teensy3x\blinky and the source files in c:\projects\Teensy3x\blinky\blinky.

The Makefile Project Wizard will allow you to enter commands for using a makefile; click Next>.

Enter the appropriate make commands for your project.  In my case, I used a makefile of blinky.mak:

Build command: make -f blinky.mak all
Clean command: make -f blinky.mak clean
Rebuild command: make -f blinky.mak clean all

Click Next>, then Finish.

At the blank VS2008 project window for blinky, right-click the Source folder in the Solution Explorer (left side of screen), then navigate to your blinky folder and select blinky.c (assuming you have downloaded the supplied files below).  I usually right-click on the Resource folder, then select blinky.mak, so I can get to the makefile quickly.

With the folders and files set up, you should be ready to build.  Right-click on the blinky icon (just below Solution 'blinky' in the Solution Explorer), then select Build.  Your build should complete and you should find a blinky.hex file in your blinky project folder.


The makefile
Here is my blinky makefile, blinky.mak.

--------------------------------------------------------------------------------------
https://github.com/Protoneer/TeensyBareMetal/blob/master/blinky.mak
------------------------------------------------------------------------

This is kind of a large file, but much of that bulk is comments.  There are a few key macros that you must set up in order to build your blinky program correctly.

The OBJECTS macro should contain all of the object files used in your project.  As a minimum, these will be your main project file (blinky.o), the sysinit.o file, and the crt0.o file.  You can add others as needed, but add them one per line and follow the format shown.

The TOOLPATH macro points to the top level of your Code Sourcery folder.

The VPATH macro should include any folders that contain source files outside of your main project folder.  In this case, VPATH contains the path to the common\ folder, because crt0.s and sysinit.c both live there and are needed by the project.

The LSCRIPT macro points to the linker script used by the project.

The default rule for converting .c files to .o files controls how C files are compiled.  There are two different compiling commands, both starting with $(CC).  Uncomment one and only one of these two lines.  If you uncomment the longer line, the included sed script will convert error messages from the cc compiler to the syntax expected by VS2008's IntelliSense; this will let you double-click on an error message and move directly to the offending line.

Similarly, there are two different assembly commands, each starting with $(AS).  Uncomment one and only one of these two lines.  You likely won't need to track errors in crt0.s, but if you do, using the longer of these lines invokes a sed script that lets you double-click on an error message and jump directly to the error in the source file.


My blinky program
Here is the source for my blinky.c file.

-----------------------------------------
https://github.com/Protoneer/TeensyBareMetal/blob/master/blinky.c
-----------------------------------------

As you can see, there isn't much to it.  The low-level initialization (in sysinit.c) is automatically invoked by crt0.s.  By the time control enters main(), the MCU clocks have been set up, the GPIO ports have clocks routed to them, and all variable initialization has been done.

This program blinks an LED with a series of eight brief pulses.  The pulses form an eight-bit hex value that corresponds to the system clock in MHz.  For example, if the system clock is 48 MHz, the LED will flash 0x30.  You will need an oscilloscope to measure the pulses.  A 0 pulse is short and a 1 pulse is long.


The files
You can find an archive of my blinky files, including my initial Teensy3x project layout, in this zip archive.


The executable
The point of the entire exercise above was to get a bare-metal LED blinker running.  The final executable, blinky.hex, takes up 2650 bytes, of which 1024 is consumed by the vector table stored at 0x0000.  To run the program, you will need the Teensy.exe downloader, available from the PJRC website.  Note that you might have to do a full Arduino install just to get this program to work.


OK, that's a wrap.  This has been a large page and I appreciate you taking the time to wade through all of this.  If you have questions or (even better) ways to improve what I have here, please drop me an email.



Home

