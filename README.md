# STM-black-pill-with-ultra-sonic-at-25HZ-rate
stm32f411 with ultra sonic reads with no blocking at 25hz rate without delay hal function
wires 
| stm32  | ultra sonic|
|--------|------------|
| 5V     | VCC        |
| GND    | GND        |
| PA 1   | TRIG       |
| PA 2   | ECHO       |


stm parameters : 
RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI

  RCC_OscInitStruct.HSIState = RCC_HSI_ON
  
  RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT
  
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON
  
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI
  
  RCC_OscInitStruct.PLL.PLLM = 8
  
  RCC_OscInitStruct.PLL.PLLN = 84
  
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2
  
  RCC_OscInitStruct.PLL.PLLQ = 4
  HCLK = 84 MHZ

Non-Blocking Architecture (EXTI + State Machine): Using HAL_Delay() blocks the CPU. 
The HC-SR04 can take up to 38ms to return a maximum-range echo. A blocking while() loop would waste millions of clock cycles.
By using an EXTI (External Interrupt) on both the rising and falling edges of the Echo pin, the CPU is free to execute other code. The interrupt simply records the start and end times.

Trigger Pulse Handling (State Machine): Even the 10µs trigger pulse is made non-blocking by checking the microsecond hardware timer in the main loop instead of using a delay.

1 MHz Hardware Timer: A timer (e.g., TIM2) is configured to tick exactly once every microsecond. This makes calculating the pulse width trivial and avoids floating-point math during the time-capture phase.
25Hz Stream Rate (40ms Period): 1 second/25Hz=40ms. 
The maximum echo time of the HC-SR04 is ~38ms. Triggering exactly every 40ms perfectly satisfies the stream rate requirement while ensuring previous acoustic echoes have dissipated before the next ping.

Timeout Recovery: If the sensor misses an echo (e.g., object out of range or wiring issue), a blocking code would freeze forever. A software timeout is implemented in the state machine to reset the sensor if no echo is received within 38ms, ensuring system reliability.
