# TMS 320 controller software
Controller software for the implementation of motor drive control on the Texas instruments TMS320 Piccolo series of microcontrollers.

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
In code composer studio you need to configure your environment variables such that the compiler is able to locate the headers from controlSUITE. 
