# STM32F103C8T6
(Hal库stm32，使用STM32CubeMX+STM32CubeIDE for VSCode插件+VSCode编写，部分程序源码来源于江协科技，个人学习总结)

2026/07/10 
led_blink：PIN_C13开漏输出高电平点亮板载蓝色LED

    HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13, GPIO_PIN_RESET);
    HAL_Delay(1000);  //延迟1000ms
    HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13, GPIO_PIN_SET);
    HAL_Delay(1000);

2026/07/12 
led_color：

    uint16_t led_stage = 0;  //定义16位无符号整数全局变量

    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, (led_stage == 0) ? GPIO_PIN_RESET : GPIO_PIN_SET); //黄
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, (led_stage == 1) ? GPIO_PIN_RESET : GPIO_PIN_SET); //红
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, (led_stage == 2) ? GPIO_PIN_RESET : GPIO_PIN_SET); //绿
    HAL_Delay(500); //延迟500ms
    led_stage++;
    if (led_stage > 2) 
    {
      led_stage = 0;
    } 

led_key：PIN_C13开漏输出高电平点亮板载蓝色LED；PIN_A9输入高电平关闭板载蓝色LED

    if(HAL_GPIO_ReadPin( GPIOA,GPIO_PIN_9) == GPIO_PIN_SET)
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

OLED_HAL(4针脚I2C版本)：PIN_B8开漏输出低电平；PIN_B9开漏输出低电平

1.添加OLED驱动源码于Drivers中
2.位于/cmake/stm32cubemx/Cmakelists.txt文件的修改(万能的豆包所给出解决方法)：

 (1).源码列表补充OLED.c

    set(MX_Application_Src
        ${CMAKE_CURRENT_SOURCE_DIR}/../../Drivers/OLED/Src/oled.c
    )

 (2).头文件检索目录补充/Drivers/OLED/Inc

    set(MX_Include_Dirs
        ${CMAKE_CURRENT_SOURCE_DIR}/../../Drivers/OLED/Inc
    )

3.在OLED.c文件中进行引脚配置

    /*引脚配置*/
    #define OLED_W_SCL(x)		HAL_GPIO_WritePin(GPIOB, GPIO_PIN_8, x) //PB8引脚
    #define OLED_W_SDA(x)		HAL_GPIO_WritePin(GPIOB, GPIO_PIN_9, x) //PB9引脚

4.在main.c中写代码

    #include "OLED.h"
    OLED_Init();		//OLED初始化
    /*OLED显示*/
    OLED_ShowChar(1, 1, 'A');				//1行1列显示字符A
    OLED_ShowString(1, 3, "HelloWorld!");	//1行3列显示字符串HelloWorld!
    OLED_ShowNum(2, 1, 12345, 5);			//2行1列显示十进制数字12345，长度为5
    OLED_ShowSignedNum(2, 7, -66, 2);		//2行7列显示有符号十进制数字-66，长度为2
    OLED_ShowHexNum(3, 1, 0xAA55, 4);		//3行1列显示十六进制数字0xA5A5，长度为4
    OLED_ShowBinNum(4, 1, 0xAA55, 16);	//4行1列显示二进制数字0xA5A5，长度为16
                                        //C语言无法直接写出二进制数字，故需要用十六进制表示

2026/07/13
sensorcount：PIN_B14外部中断模式，下降沿触发检测(高电平到低电平变化)

1.main.h文件:

    #include "stm32f1xx_it.h"

2.stm32f1xx_it.h文件:

    uint16_t GetCounter(void);  //函数声明

3.stm32f1xx_it.c文件:

    uint16_t count; //定义全局变量
    if(__HAL_GPIO_EXTI_CLEAR_FLAG(GPIO_PIN_14))
      {
      HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_14);
      if (HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_14) == GPIO_PIN_SET) ////确认引脚电平，防止遮光板放下+1 拿开后也+1
        {
        count++;
        }
      }
    uint16_t GetCounter(void)
    {
    return count;
    }

4.main.c文件:

    OLED_Init();		//OLED初始化
    OLED_Clear();   //OLED清屏
    OLED_ShowString(1, 1, "count:");	//1行1列显示字符串count:
    while (1)
    {
      OLED_ShowNum(2, 1, GetCounter(), 5);			//2行1列显示十进制数字GetCount()，长度为5
    }

2026/07/14
encoder：

    if(__HAL_GPIO_EXTI_GET_FLAG(GPIO_PIN_0))
    {
        if(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_0) == 0)
        Count --;
    }
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_0);

    if(__HAL_GPIO_EXTI_GET_FLAG(GPIO_PIN_1))
    {
        if(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_1) == 0)
        Count ++;
    }
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_1);

2026/07/15
PWM mode 1向上计数模式下：计数器数值 CNT < CCR（Pulse 值）时，引脚输出有效电平；CNT ≥ CCR 时输出无效电平
pwm_oled:TIM2;Clock Source:Internal Clock;Channel1:PWM Generation CH1;PSC(预分频器):720-1;ARR(自动重装器):100-1;PWM MODE 1

    HAL_TIM_PWM_Start(&htim2,TIM_CHANNEL_1);
    while (1)
    {
         for(int i = 0;i<100;i++)
        {
            __HAL_TIM_SET_COMPARE(&htim2,TIM_CHANNEL_1,i);  //CCR值由0增加到100
            HAL_Delay(10);
        }
        for(int i = 0;i<100;i++)
        {
             __HAL_TIM_SET_COMPARE(&htim2,TIM_CHANNEL_1,100-i); //CCR值由100减小到0
            HAL_Delay(10);
        }
    }

servo:TIM2;Clock Source:Internal Clock;Channel2:PWM Generation CH2;PSC(预分频器):720-1;ARR(自动重装器):20000-1;PWM MODE 1;CCR(初始捕获/比较器的值):500

    uint16_t Angle;
    
    OLED_Init();
    OLED_Clear();
    OLED_ShowString(1,1,"Angle:");
    HAL_TIM_PWM_Start(&htim2,TIM_CHANNEL_2);
    while (1)
    {
        if(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_1) == GPIO_PIN_RESET)
        {
            //按键消抖
            HAL_Delay(20);
            while (HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_1) == GPIO_PIN_RESET);
            HAL_Delay(20);
            Angle += 30;
            if (Angle > 180)
            {
              Angle = 0;
            }
        }
        __HAL_TIM_SET_COMPARE(&htim2,TIM_CHANNEL_2,Angle * (htim2.Init.Period + 1) / 1800  + (htim2.Init.Period + 1) / 40 );    //动态调整CCR的值
        OLED_ShowNum(2,1,Angle,3);
    }

2026/07/16
pwm_motor:TIM2;Clock Source:Internal Clock;Channel3:PWM Generation CH3;PSC(预分频器):72-1;ARR(自动重装器):100-1;PWM MODE 1;CCR(STM32CubeMX里设置的初始CCR的值):20

    int16_t Speed = 0;
    void SetSpeed(int8_t Speed)
    {
        if(Speed >= 0)
        {
            HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);
            HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
            __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, Speed);    //电机正转
        }
        else 
        {
            HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_RESET);
            HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
            __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, -Speed);   //电机反转
        }
    }

    OLED_Init();
    HAL_TIM_PWM_Start(&htim2,TIM_CHANNEL_3);
    OLED_Clear();
    OLED_ShowString(1,1,"Speed:");
    while (1)
    {
        if(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_1) == GPIO_PIN_RESET)
        {
            HAL_Delay(20);
            while(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_1) == GPIO_PIN_RESET);
            HAL_Delay(20);
            Speed += 1;
            if(Speed > 2)
            {
                Speed = -2;
            }
        }
        SetSpeed(Speed * 50);
        OLED_ShowSignedNum(2,1,Speed*50,3);
    }

2026/07/17
freq_detect:TIM2_CH1产生PWM信号:TIM3:Reset Mode,TI1FP1,Internal Clock,Channel1:Input Capture direct mode,开启NVIC

    int32_t freq;
    void (HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim))
    {
        if(htim -> Instance == TIM3)
        {
            uint32_t capture = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1) +1;   //HAL_TIM_ReadCapturedValue读CCR1的值
            freq = 1000000 / capture;
        }
    }

    OLED_Init();
    HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
    HAL_TIM_IC_Start_IT(&htim3, TIM_CHANNEL_1);
    OLED_Clear();
    OLED_ShowString(1, 1, "Freq:000000Hz");
    while (1)
    {
        OLED_ShowNum(1, 6, freq, 5);
    }

    





2026/07/18
pwmi_duty:TIM2_CH1产生PWM信号;TIM3:Reset Mode,TI1FP1,Internal Clock,Channel1:Input Capture direct mode通道1直通输入捕获(引脚电平跳变时，硬件将当前 CNT 锁存存入 CCR1 捕获寄存器;同时该信号输出 TI1FP1，触发从模式复位，清零CNT),Channel2:Input Capture indirect mode通道2间接输入捕获(通道2复用通道1的滤波信号 TI1FP1 作为捕获输入，不用外接引脚;同一个输入信号既给CH1直通捕获,又给CH2间接捕获;可分别配置两个通道捕获边沿,一次输入信号就能同时记录上升沿、下降沿时刻,实现脉宽测量。),开启NVIC,PSC:72-1,Channel1:Rising Edge(direct),Channel2:Falling Edge(indirect)

    int32_t freq;
    uint32_t duty;
    uint32_t capture;
    void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
    {
        if(htim -> Instance == TIM3)
        {
            //上升沿出发中断
            if(htim -> Channel == HAL_TIM_ACTIVE_CHANNEL_1)
            {
                capture = HAL_TIM_ReadCapturedValue(htim,TIM_CHANNEL_1) + 1;    //HAL_TIM_ReadCapturedValue读CCR1的值
                freq = 1000000 / capture;
            }
            //下降沿触发中断
            else if(htim -> Channel == HAL_TIM_ACTIVE_CHANNEL_2)
            {
                uint32_t capture2 = HAL_TIM_ReadCapturedValue(htim,TIM_CHANNEL_2) + 1;
                duty = capture2 * 100 / capture;
            }
        }
    }

    OLED_Init();
    HAL_TIM_PWM_Start(&htim2,TIM_CHANNEL_1);
    HAL_TIM_IC_Start_IT(&htim3,TIM_CHANNEL_1);  //打开输入捕获周期
    HAL_TIM_IC_Start_IT(&htim3,TIM_CHANNEL_2);  //打开输入捕获占空比
    OLED_Clear();
    OLED_ShowString(1,1,"Freq:000000HZ");
    OLED_ShowString(2,1,"Duty:000%");
    while (1)
    {
        OLED_ShowSignedNum(1,6,freq,5);
        OLED_ShowSignedNum(2,6,duty,2);
    }





2026/07/19
encoder_detect:TIM3;Encoder Mode;Encoder Mode TI1 and TI2

    int16_t GetEncoderSpeed()
    {
        int16_t tmp;
        tmp = __HAL_TIM_GET_COUNTER(&htim3); //读取CNT计数器的值
        __HAL_TIM_SET_COUNTER(&htim3, 0);   //CNT计数器的值清零
        return tmp;
    }

    OLED_Init();
    HAL_TIM_Encoder_Start(&htim3, TIM_CHANNEL_ALL);
    OLED_Clear();
    OLED_ShowString(1, 1, "SPD:");
    while (1)
    {
        OLED_ShowSignedNum(2, 5, GetEncoderSpeed(), 5);
        HAL_Delay(1000);
    }
    
2026/07/22
adc:ADC1 Mode:IN0;ADC PSC 12Mhz

    OLED_Init();
    OLED_Clear();
    HAL_ADCEx_Calibration_Start(&hadc1);    //ADC1校准
    OLED_ShowString(1, 1, "Value:");
    OLED_ShowString(2, 1, "Voltage:0.00V");
    while (1)
    {
        HAL_ADC_Start(&hadc1);  //开启ADC
        HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY); //等待转换完成
        value = HAL_ADC_GetValue(&hadc1); //获取结果
        voltage = (float)value/4095.0*3.3;
        OLED_ShowNum(1, 10, value, 4);
        OLED_ShowNum(2, 9, (uint32_t)voltage, 1);
        OLED_ShowNum(2, 11, (uint16_t)(voltage*100)%100, 2);
        HAL_Delay(1000);
    }

multi_adc:ADC1 Mode:IN0,IN1,IN2,IN3;ADC Settings:Scan Conversion Mode Enabled,Discontinuous Conversion Mode Enabled;ADC_Regular_ConversionMode:Numble of Conversion 4,Rank1~4:Channel0~3

    uint16_t values[4] = {0}; //用于存放ADC数据
    uint8_t i;

    OLED_Init();
    OLED_Clear();
    HAL_ADCEx_Calibration_Start(&hadc1);
    OLED_ShowString(1, 1, "V1:");
    OLED_ShowString(2, 1, "V2:");
    OLED_ShowString(3, 1, "V3:");
    OLED_ShowString(4, 1, "V4:");
    while (1)
    {
        for (i = 0; i < 4; i++)
        {
            HAL_ADC_Start(&hadc1);
            if(HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY) == HAL_OK)
            {
                values[i] = HAL_ADC_GetValue(&hadc1);
            }    
        }
        OLED_ShowNum(1, 5, values[0], 4);
        OLED_ShowNum(2, 5, values[1], 4);
        OLED_ShowNum(3, 5, values[2], 4);
        OLED_ShowNum(4, 5, values[3], 4);
        HAL_Delay(1000);
    }    

dma_m2m:DMA1 default Settings

    OLED_Init();
    OLED_Clear();
    uint8_t DataA[] = {0x01,0x02,0x03,0x04};
    uint8_t DataB[] = {0,0,0,0};
    while (1)
    {
        HAL_DMA_Start(&hdma_memtomem_dma1_channel1, (uint32_t)&DataA, (uint32_t)&DataB, 4); //启动DMA
        HAL_DMA_PollForTransfer(&hdma_memtomem_dma1_channel1, HAL_DMA_FULL_TRANSFER, HAL_MAX_DELAY);  //等待传输完成
        for (uint16_t i = 0; i<4; i++)
        {
            DataA[i]++;
        }
        for (uint16_t i = 0; i<4; i++)
        {
            OLED_ShowHexNum(1, 1+3*i, DataA[i], 2);
        }
        for (uint16_t j = 0; j<4; j++)
        {
            OLED_ShowHexNum(3, 1+3*j, DataA[j], 2);
        }
        HAL_Delay(50);
    }

dma_adc:ADC1 Mode:IN0,IN1,IN2,IN3;ADC Settings:Scan Conversion Mode Enabled,Continuous Conversion Mode Enabled;ADC_Regular_ConversionMode:Numble of Conversion 4,Rank1~4:Channel0~3,55.5Cycles(提高采样精度);ADC DMA Settings:Circular,Other default

    uint16_t value[4];

    OLED_Init();
    HAL_ADCEx_Calibration_Start(&hadc1);
    OLED_Clear();
    HAL_ADC_Start_DMA(&hadc1, (uint32_t*)value, 4);
    while (1)
    {
        OLED_ShowNum(1, 1, value[0], 5);
        OLED_ShowNum(2, 1, value[1], 5);
        OLED_ShowNum(3, 1, value[2], 5);
        OLED_ShowNum(4, 1, value[3], 5);
        HAL_Delay(1000);
    }
