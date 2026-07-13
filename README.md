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

