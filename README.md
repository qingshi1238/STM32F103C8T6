# STM32F103C8T6
2026/07/10 
led_blink：PIN_C13开漏输出高电平点亮板载蓝色LED

    HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13, GPIO_PIN_RESET);
    HAL_Delay(1000);  //延迟1000ms
    HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13, GPIO_PIN_SET);
    HAL_Delay(1000);

2026/07/12 
led_color：

    uint16_t led_stage = 0;  //定义16位无符号整数变量
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, (led_stage == 0) ? GPIO_PIN_RESET : GPIO_PIN_SET); // Yellow
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, (led_stage == 1) ? GPIO_PIN_RESET : GPIO_PIN_SET); // Red
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, (led_stage == 2) ? GPIO_PIN_RESET : GPIO_PIN_SET); // green
    HAL_Delay(500); //延迟500ms
    led_stage++;
    if (led_stage > 2) 
    {
      led_stage = 0;
    } 


led_key：PIN_C13开漏输出高电平点亮板载蓝色LED；PIN_A9输入高电平关闭板载蓝色LED
    If(HAL_GPIO_ReadPin( GPIOA,GPIO_PIN_9) == GPIO_PIN_SET)
    {
      HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
    }
    else
    {
      HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
    }

buzzer：PIN_B12推挽输出高电平,GPIO上拉；蜂鸣器低电平响

    HAL_GPIO_WritePin(GPIOB,GPIO_PIN_12,RESET);
    HAL_Delay(500);  //延迟500ms
    HAL_GPIO_WritePin(GPIOB,GPIO_PIN_12,SET);
    HAL_Delay(500);

buzzer & LDR：PIN_B12推挽输出高电平,GPIO上拉；蜂鸣器低电平响；光敏电阻光线越强，下拉电阻变大，输出电压变小，输出0，蜂鸣器响

    if(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_13) == GPIO_PIN_RESET)
    {
      HAL_GPIO_WritePin(GPIOB,GPIO_PIN_12,RESET);
    }
    else
    {
      HAL_GPIO_WritePin(GPIOB,GPIO_PIN_12,SET);
    }


