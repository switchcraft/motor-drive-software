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
Further down the header defines a struct which has the same size as the GPIO control registers. Please 

### Linker command files
The directory "controlSUITE/device_support/f2802x/v230/f2802x_common/cmd" contains varius command files to tell the linker where in memory the varius peripherals are located.
