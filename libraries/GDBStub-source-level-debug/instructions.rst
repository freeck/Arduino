**Instructions command line debugging ESP8266**

+----+----------------+------------------------------------------------------------------------+-----------+--------+-------------+
|    | Author		  | Fred Kuijper														   | Version   | 0.1    | 25-1-2018   |
+====+================+========================================================================+===========+========+=============+
|    |                | Translation to English                                                 | Version   | 0.2    | 1-02-2018   |
+----+----------------+------------------------------------------------------------------------+-----------+--------+-------------+
|    |                | Minor modifications                                                    | Version   | 0.21   | 4-2-2018    |
+----+----------------+------------------------------------------------------------------------+-----------+--------+-------------+

Status: under construction!

Scope 
======
The scope is limited to preparing and testing a command-line gdb-debugging session for the esp8266 processor. It does not contain an instruction on how to install an IDE for the esp8266 nor the tool chain itself.
The instruction is based on a Windows-environment. The primary source of the tools and gdbstub-code is http://gnutoolchains.com/download/.
A few modifications have been made to the sources: some IRAM-instructions have placed to FLASH, and some assembly-functions have been alignment to 4 bytes (in order to enable the code being called via
a long-call).

Command line debugging
======================
The next steps should be followed to start command line debugging.
1: Download the files from github.
*The directory should look like this:*

.. figure:: gdbstub-lib.png

Directory “\ *bin*\ ” contains all the files necessary to execute a
command line debug-session.:

.. figure:: gdbstub-dir.png
Copy the files of directory “\ *bin”* to a location of your choice.
Copy the library in directory “\ *gdbstub*\ ” to a location where you normally put your libraries.
**Mind you**: The original files from *VisualGDB* have been modified in order to reduce the IRAM-usage, and also have set the some #defines in file *gdbstub-cfg.h*.

2: Prepare your own sketch or use the example-sketch provided by the repository. In case you use your own sketch, the following lines have to be added.

 1.Add #include "gdbstub.h" 
 
 2.Add to function setup():
   Serial.begin(115200);
   
   gdbstub_init();

**Mind you**: After this **Serial.printf()** doesn’t work anymore as
gdbstub\_init() takes over the UART-interrupt, instead use function
**ets\_printf** for output of messages!

4: Build the example app using compiler/linker options: “-\ *ggdb –Og
–mlongcalls*\ ”.
https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html#Debugging-Options.
After a new build, be sure to copy the latest application elf-file to
the directory where the bin-files reside.
The Arduino\_IDE compiler/linker compiles with options: “\ *-Os -g
–mlongcalls”* (*I do not know how and where to set these options*) , for
“Eclipse-Sloeber “ set the options to “–\ *ggdb –Og –mlongcalls*\ ” ,
*“–Og”* generates easier to debug code (see
`*https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html* <https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html>`__
*). *

5: File “\ *gdbcmds”* is the configuration-file containing the arguments for starting the debug-session:
 1.set remote hardware-breakpoint-limit 1
 
 2.set remote hardware-watchpoint-limit 1
 
 3.set debug xtensa 4
 
 4.target remote `\\\\.\\COM9 <file:///\\.\COM9>`__ (**change to your own com-port**)
 
 5.file blink.ino.elf (**the application of choice, add complete path if elf-file resides somewhere else**)
 
 6.thb loop (or any location you want your program to start the debug-session).

6: Open a command shell and go to the working directory where *the files
xtensa-lx106-elf-gdb* and *gdbcmds* reside. Copy the latest ELF-file to
this directory.

7: Then download the application and verify that the debug-stub is actually
present and has started: 

.. figure:: gdbstub-start.png

You should see: **“$ST05#b9”** on the debug-port of the esp8266.

8: Then execute command: “\ *xtensa-lx106-elf-gdb.exe -x gdbcmds*\ ”. If
everything goes well the application will start and stop at location
“\ *loop*\ ”. 
Output on the debug-port:

.. figure:: gdbstub-comport-output.png

Output from the target:

.. figure:: gdbstub-breakpoint1.png

Then execute command “\ *info break*\ ”:

.. figure:: gdbstub-info-break.png
And the breakpoints defined in file *gdbcmds* are shown.

9: Continue the program by entering “\ *continue*\ ”

.. figure:: gdbstub-thb-loop.png
After 1000 milliseconds the program will stop at function “\ *loop*\ ”.

*Add a watchpoint*
1: Define a *watchpoint* and an *expression*
2: Continue executing and wait.
*Condition reached*

For more extensive info about *command-line gdb* I refer to
http://sourceware.org/gdb/onlinedocs/gdb/index.html#SEC_Contents

This was it! I hope this may inspire you to use source level
debugging. I personally prefer and would recommend using Eclipse-Sloeber
as IDE.

Issues, tips and tricks
=======================

1. Put the code to be debugged preferably in IRAM, you can then put 
   multiple breakpoints in a function (the esp8266 has only one hardware
   breakpoint).

2. Since esp8266 has only 1 hardware-breakpoint, it is useful to define a
   so-called hardware temporary breakpoint (*thb*). *thb* or *hb*
   can also be used to set a breakpoint in Flash code. A temporary
   breakpoint is deactivated after each hit, so a next hardware
   breakpoint may be set.

3. Always check whether the target actually contains *gdbstub*. Start a
   terminal program (for example *RealTerm*), connect with the com port,
   reset the target. If all goes well, a message from *gdbstub* will
   appear on the screen: **“$T05#B9**\ ”. If not, the application is not
   properly built.

4. Sometimes the debugger doesn’t stop at a certain location: this may
   be caused by the compiler’s optimizer, the optimizer could generate
   inline code (see the function calculate () as shown below)! To
   minimize these effects compile with flag “–Og” .

.. figure:: gdbstub-watchpoint-setcondition-reached.png

5. Function **ets\_printf()** and **Serial.printf()** behave
   differently. The serial-IO for the Arduino-IDE is based on *class
   Serial* , the debugger uses the standard-IO library...

   **Serial.prinf()** is disabled by *gdbstub*, so use **ets\_printf()**
   instead.

References
==========

1. https://github.com/espressif/esp-gdbstub

2. https://github.com/esp8266/Arduino/tree/master/libraries/GDBStub/

3. https://espressif.com/sites/default/files/documentation/esp8266_reset_causes_and_common_fatal_exception_causes_en.pdf

4. https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html

5. http://sourceware.org/gdb/onlinedocs/gdb/index.html#SEC_Contents

6. https://visualgdb.com/tutorials/esp8266/gdbstub/

7. http://gnutoolchains.com/download/
