# TMS 320 controller software
Controller software for the implementation of motor drive control on the Texas instruments TMS320 Piccolo series of microcontrollers.

Our development platform is built upon the TMS320F2802x. Datasheet: <http://www.ti.com/lit/ds/symlink/tms320f28027.pdf>.

## Development environment
There may be other (open source) approches to development for the TMS320 Piccolo series of microcontrollers, but we are useing the officially supportet software from Texas instruments:
* Code composer studio (CCS) (<http://processors.wiki.ti.com/index.php/Download_CCS>)
* controlSUITE (<http://www.ti.com/tool/controlsuite>)

### controlSUITE
Control suite is a huge collection of different libraries and example code. We are only using a small portion of this library (i.e. the hardware spesific parts), as we typically prefer to write our own functions for the more general control problems.

Under GNU/Linux you should use Wine to install controlSUITE. There will be issues, but we only need to extract some text files (i.e. header files):

```
#include "f2802x_common/include/DSP28x_Project.h"

#include "f2802x_common/include/clk.h"
#include "f2802x_common/include/gpio.h"
#include "f2802x_common/include/pll.h"
#include "f2802x_common/include/wdog.h"

#include "f2802x_common/include/flash.h"
#include "f2802x_common/include/pie.h"
#include "f2802x_common/include/adc.h"
#include "f2802x_common/include/sci.h"
#include "f2802x_common/include/sci_io.h"

/*
 * Primary include file for the F2802x controller.
 */
#include "f2802x_headers/include/F2802x_Device.h"
```

### Code composer studio
In code composer studio you need to configure a environment variable such that the compiler is able to locate the headers from controlSUITE.

We are using a variable named CSUITE, that is pointing to the install directory of controlSUITE.

## How does it work?
As most (all?) microcontrollers, the piccolo series relies on data registers to control the various hardware peripherals. The addresses for the various registers are available in the datasheet for a given controller, but it is typically much simpler to use the names defined in the supplied header files, rather than using the addresses directly.

As an example, the registers intended for GPIO control starts at address 0x6F80, and this is defined in the header file "gpio.h" as:
```
#define  GPIO_BASE_ADDR        (0x00006F80)
```
Further down the header defines a struct which has the same size as the GPIO control registers. 

## Linker command files
The linker command files may contain any parameter that is passed to the linker. The most important however is the assignment of the various memory locations in the controller.

The directory "controlSUITE/device_support/f2802x/v230/f2802x_common/cmd" contains varius command files to tell the linker where in memory the varius input sections (e.g. .text, .bss, .data) should be located. Additionally the directory: "controlSUITE/device_support/f2802x/v230/f2802x_headers/cmd" contains the linker files used to link the structures used for communication with the hardware pheripherals.

### The MEMORY directive
First the "MEMORY" directive is used to assign names to the various memory locations. It associates various names with a origin address, and a length of the memory location.

```
MEMORY
{
PAGE 0:    /* Program Memory */
           /* Memory (RAM/FLASH/OTP) blocks can be moved to PAGE1 for data allocation */

   PRAML0      : origin = 0x008000, length = 0x000800     /* on-chip RAM block L0 */
   OTP         : origin = 0x3D7800, length = 0x000400     /* on-chip OTP */
   FLASHD      : origin = 0x3F0000, length = 0x002000     /* on-chip FLASH */
   FLASHC      : origin = 0x3F2000, length = 0x002000     /* on-chip FLASH */
   FLASHA      : origin = 0x3F6000, length = 0x001F80     /* on-chip FLASH */
   CSM_RSVD    : origin = 0x3F7F80, length = 0x000076     /* Part of FLASHA.  Program with all 0x0000 when CSM is in use. */
   BEGIN       : origin = 0x3F7FF6, length = 0x000002     /* Part of FLASHA.  Used for "boot to Flash" bootloader mode. */
   CSM_PWL_P0  : origin = 0x3F7FF8, length = 0x000008     /* Part of FLASHA.  CSM password locations in FLASHA */

   IQTABLES    : origin = 0x3FE000, length = 0x000B50     /* IQ Math Tables in Boot ROM */
   IQTABLES2   : origin = 0x3FEB50, length = 0x00008C     /* IQ Math Tables in Boot ROM */
   IQTABLES3   : origin = 0x3FEBDC, length = 0x0000AA      /* IQ Math Tables in Boot ROM */

   ROM         : origin = 0x3FF27C, length = 0x000D44     /* Boot ROM */
   RESET       : origin = 0x3FFFC0, length = 0x000002     /* part of boot ROM  */
   VECTORS     : origin = 0x3FFFC2, length = 0x00003E     /* part of boot ROM  */

PAGE 1 :   /* Data Memory */
           /* Memory (RAM/FLASH/OTP) blocks can be moved to PAGE0 for program allocation */
           /* Registers remain on PAGE1                                                  */

   BOOT_RSVD   : origin = 0x000000, length = 0x000050     /* Part of M0, BOOT rom will use this for stack */
   RAMM0       : origin = 0x000050, length = 0x0003B0     /* on-chip RAM block M0 */
   RAMM1       : origin = 0x000400, length = 0x000400     /* on-chip RAM block M1 */
   DRAML0      : origin = 0x008800, length = 0x000800     /* on-chip RAM block L0 */
   FLASHB      : origin = 0x3F4000, length = 0x002000     /* on-chip FLASH */
}
```

### The SECTIONS directive
The SECTIONS directive is used to plase the code and data from the object files in the correct memory locations.

```
SECTIONS
{

   /* Allocate program areas: */
   .cinit              : > FLASHA | FLASHC | FLASHD,       PAGE = 0
   .pinit              : > FLASHA | FLASHC | FLASHD,      PAGE = 0
   .text               : >> FLASHA | FLASHC | FLASHD,       PAGE = 0
   codestart           : > BEGIN        PAGE = 0
   ramfuncs            : LOAD = FLASHA,
                         RUN = PRAML0,
                         LOAD_START(_RamfuncsLoadStart),
                         LOAD_SIZE(_RamfuncsLoadSize),
                         RUN_START(_RamfuncsRunStart),
                         PAGE = 0

   csmpasswds          : > CSM_PWL_P0   PAGE = 0
   csm_rsvd            : > CSM_RSVD     PAGE = 0

   /* Allocate uninitalized data sections: */
   .stack              : > RAMM0        PAGE = 1
   .ebss               : > DRAML0       PAGE = 1
   .esysmem            : > DRAML0       PAGE = 1
   .sysmem             : > DRAML0       PAGE = 1
   .cio                : >> RAMM0 | RAMM1 | DRAML0       PAGE = 1

   /* Initalized sections go in Flash */
   /* For SDFlash to program these, they must be allocated to page 0 */
   .econst             : > FLASHA       PAGE = 0
   .switch             : > FLASHA       PAGE = 0

   /* Allocate IQ math areas: */
   IQmath              : > FLASHA       PAGE = 0            /* Math Code */
   IQmathTables        : > IQTABLES,    PAGE = 0, TYPE = NOLOAD

   
   /* .reset is a standard section used by the compiler.  It contains the */
   /* the address of the start of _c_int00 for C Code.   /*
   /* When using the boot ROM this section and the CPU vector */
   /* table is not needed.  Thus the default type is set here to  */
   /* DSECT  */
   .reset              : > RESET,      PAGE = 0, TYPE = DSECT
   vectors             : > VECTORS     PAGE = 0, TYPE = DSECT

}
```
