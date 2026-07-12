# STM32F103C8T6
2026/07/10 led_blink：C13 OutPut Open Drain High Level light up LED_Blue

    HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13, GPIO_PIN_RESET);
    HAL_Delay(1000);
    HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13, GPIO_PIN_SET);
    HAL_Delay(1000);

2026/07/12 led_color：

    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, (led_stage == 0) ? GPIO_PIN_RESET : GPIO_PIN_SET); // Yellow
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, (led_stage == 1) ? GPIO_PIN_RESET : GPIO_PIN_SET); // Red
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, (led_stage == 2) ? GPIO_PIN_RESET : GPIO_PIN_SET); // green
    HAL_Delay(500); // Delay for 0.5 second
    led_stage++;
    if (led_stage > 2) 
    {
      led_stage = 0;
    } 


2026/07/12 led_key：C13 OutPut Open Drain High Level light up LED_Blue,A9 InPut High Level turns off LED_Blue

    If(HAL_GPIO_ReadPin( GPIOA,GPIO_PIN_9) == GPIO_PIN_SET)
    {
      HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
    }
    else
    {
      HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
    }
