STM32 Nucleo-F446RE: EXTI button interrupt toggling an LED (bare-metal)

WHAT IT DOES
Pressing the user button B1 (PC13) triggers an external interrupt (EXTI line 13).
The interrupt handler toggles the user LED on PA5. main() is an empty loop:
all the work happens in the ISR.

WHAT IT DEMONSTRATES
- Register-level C, no HAL and no CMSIS device headers
  (addresses are defined by hand as volatile pointers).
- GPIO setup: PA5 as output, PC13 as input (GPIOx_MODER).
- Routing a pin to an EXTI line through SYSCFG_EXTICR4, with the SYSCFG clock enabled.
- EXTI configuration: falling-edge trigger (EXTI_FTSR), unmask line 13 (EXTI_IMR).
- NVIC: enabling IRQ 40 (EXTI15_10) through NVIC_ISER1 bit 8.
- ISR: check the pending bit, toggle the LED, clear the flag.
  EXTI_PR is write-1-to-clear, so the flag is cleared with "=" and not "|=",
  otherwise other pending lines would be cleared as well.

HARDWARE
- Board: NUCLEO-F446RE (MCU STM32F446RETx)
- LED: PA5 (user LED LD2)
- Button: PC13 (user button B1, active low)

INTERRUPT PATH
PC13 -> SYSCFG_EXTICR4 -> EXTI line 13 (FTSR, IMR) -> NVIC IRQ 40 -> EXTI15_10_IRQHandler

BUILD AND RUN
1. Open the project in STM32CubeIDE (File > Open Projects from File System).
2. Build, then flash with the Debug configuration.
3. Press B1: the LED changes state on each press.

STATUS
Tested on a NUCLEO-F446RE board: 10 button presses, the LED toggled as expected.

KNOWN LIMITATIONS
- No software debounce on the button.
- The LED output register (GPIOA_ODR) is toggled with a read-modify-write.
  This is safe here because only the ISR writes to it. If main() also wrote to
  the same register, an interrupt could cause a lost update.
