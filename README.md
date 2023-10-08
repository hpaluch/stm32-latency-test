# STM32 Interrupt latency test

Here is my quick Interrupt latency test
for [STM NUCLEO-F767ZI][STM NUCLEO-F767ZI] development board
with Cortex-M7 CPU. 

Quick Results - are quite good:
- 1st channel (Yellow) is just slowly blinking LED LD1 on PB0 port
  at toggle rate 10 Hz (frequency 5 Hz)
- 2nd channel tries to quickly toggle PE0 GPIO pin in main loop:
  ```c
  while (1)
  {
	  HAL_GPIO_TogglePin(GPIOE, GPIO_PIN_0); // Toggle PE0
  }
  ```
- where 1st channel is trigger and also Interrupt handler.

![STM32 Latency scope](digilentad2/STM32-LED-Timer-And-Poll-HF.gif)

And Workspace file [digilentad2/STM32-LED-Timer-And-Poll-HF.dwf3work](digilentad2/STM32-LED-Timer-And-Poll-HF.dwf3work)

From Scope (Digilent Analog Discovery 2) we can see that:
- PE0 GPIO Output frequency is around 10 Mhz - so toggle rate is
  impressive 20 MHz
- Servicing interrupt takes around 1.32 micro-seconds - which is very good.

Why GPIO is not faster?
* according to [DS11532][DS11532] page 49:

  > A fast I/O handling allows a maximum I/O toggling up to 108 MHz.

* but looking to [RM0410][RM0410] page 91,
  Table 7. Number of wait states according to CPU clock (HCLK) frequency

  The Program FLASH is capable to handle only up to 30 MHz without wait states.
  For 216 MHz there are at least 7 Wait states (8 CPU cycles)

* To solve this problem one has to:
  - move GPIO code from slow Flash to TC (Time Critical) RAM
  - access directly GPIO registers (this example calls HAL routines which
    has significant overhead)

* However my 100 MHz scope is unable to handle such speed so I have no plan to try it.

# Setup

Required Hardware:
* [STM NUCLEO-F767ZI][STM NUCLEO-F767ZI] development board with Cortex-M7 CPU. 
  Ordered [STM NUCLEO-F767ZI - Amazon.de][STM NUCLEO-F767ZI - Amazon.de]

Recommended Hardware:
* `Digilent Analog Discovery 2` scope + `WaveForms 3` Software

Required Software:
* [STM32CubeF7][STM32CubeF7] Firmware package required to build this project.
  You need to download and unpack `en.stm32cubef7_v1-17-0.zip` first
  that download and unpack *to same location - overwrite files* patch
  archive `en.stm32cubef7-v1-17-1.zip`
* [System Workbench for STM32][System Workbench for STM32] development IDE

# Build

* First copy modified [Src/main.c](Src/main.c) to your
  unpacked CubeF7 tree, for example:
  ```
  c:\Ac6\STM32Cube_FW_F7_V1.17.0\Projects\STM32F767ZI-Nucleo\Examples\TIM\TIM_TimeBase\Src
  ```
* Then run AC6 IDE
* Click on Import Existing Projects ...
* select Path like:
  ```
  c:\Ac6\STM32Cube_FW_F7_V1.17.0\Projects\STM32F767ZI-Nucleo\Examples\TIM\TIM_TimeBase\SW4STM32\STM32F767ZI-Nucleo
  ```
*  Warning! All these examples use same project name `STM32F767ZI-Nucleo`,
   however Ac6 insists that Project name must be unique. So I recommend to rename
   this project right after import...

To generate assembler listing - ensure that you already build .elf
binary and try:
```cmd
cd Debug
c:\Ac6\SystemWorkbench\plugins\fr.ac6.mcu.externaltools.arm-none.win32_1.17.0.201812190825\tools\compiler\arm-none-eabi\bin\objdump.exe ^
  -S STM32F767ZI-Nucleo.elf > STM32F767ZI-Nucleo.lst
```

# Resources

* Please see my [Getting started with ST NUCLEO F767ZI Board][Getting started with ST NUCLEO F767ZI Board]
  for introduction.

[DS11532]: https://www.st.com/resource/en/datasheet/stm32f767zi.pdf
[RM0410]: https://www.st.com/resource/en/reference_manual/rm0410-stm32f76xxx-and-stm32f77xxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf
[STM32CubeF7]: https://www.st.com/en/embedded-software/stm32cubef7.html
[System Workbench for STM32]: http://www.openstm32.org/System%2BWorkbench%2Bfor%2BSTM32
[STM32CubeMX]: https://www.st.com/content/st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-configurators-and-code-generators/stm32cubemx.html

[STM NUCLEO-F767ZI - Amazon.de]: https://www.amazon.de/dp/B072MMZZBK/
[STM NUCLEO-F767ZI]: https://www.st.com/content/st_com/en/products/evaluation-tools/product-evaluation-tools/mcu-eval-tools/stm32-mcu-eval-tools/stm32-mcu-nucleo/nucleo-f767zi.html
[Getting started with ST NUCLEO F767ZI Board]: https://github.com/hpaluch/hpaluch.github.io/wiki/Getting-started-with-ST-NUCLEO-F767ZI-Board
[STM32CubeF7]: https://www.st.com/en/embedded-software/stm32cubef7.html
[Getting started with ST NUCLEO F767ZI Board]: https://github.com/hpaluch/hpaluch.github.io/wiki/Getting-started-with-ST-NUCLEO-F767ZI-Board

