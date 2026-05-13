# 一、OLED
## OLED驱动库移植
* ####  拷贝`oled.c`、`font.c`、`oled.h`、`font.h` 拷贝到工程
* ####  工程引用头文件`#include "oled.h"`

* #### 配置 CMakeLists.txt
```cmke
    # Add user sources here
    Core/Src/oled.c
    Core/Src/font.c

    # Add user defined include paths
    Core/Inc
```

* #### 修改I2C设备句柄
  * 在初始化代码里，oled.c中将原有 `&hi2c2` 改成自己工程配置的 `I2C1`：

* #### 低版本GCC兼容修复
  * GCC 10及以下不支持链接脚本 `READONLY` 关键字，会链接报错：  
打开 `STM32F103XX_FLASH.ld`：  
找到这个关键字，全部删除
```ld
.ARM.extab (READONLY) :
.ARM (READONLY) :
```

* #### 中文字体显示
  * 用如18x18的`其他大小字体`时
    * 取模之后直接全部复制粘贴在`font.c`
    * 然后再`font.h`中声明 `extern const Font font18x18` 
  *  用`16x16字体`时
     * 将在字体取模后，将中间字体复制进去
     * 将下面`const`中的第四个参数改为`sizeof(zh16x16/36)` 
```c
const uint8_t zh16x16[][36] = {  //这里时36，待会就÷36
/* 0 波 */ {0xe6,0xb3,0xa2,0x00,0x10,0x60,0x02,0x0c,0xc0,0x00,0xf8,0x88,0x88,0x88,0xff,0x88,0x88,0xa8,0x18,0x00,0x04,0x04,0x7c,0x03,0x80,0x60,0x1f,0x80,0x43,0x2c,0x10,0x28,0x46,0x81,0x80,0x00,},
/* 1 特 */ {0xe7,0x89,0xb9,0x00,0x40,0x3c,0x10,0xff,0x10,0x10,0x40,0x48,0x48,0x48,0x7f,0x48,0xc8,0x48,0x40,0x00,0x02,0x06,0x02,0xff,0x01,0x01,0x00,0x02,0x0a,0x12,0x42,0x82,0x7f,0x02,0x02,0x00,},
/* 2 律 */ {0xe5,0xbe,0x8b,0x00,0x00,0x10,0x88,0xc4,0x33,0x10,0x54,0x54,0x54,0xff,0x54,0x54,0x7c,0x10,0x10,0x00,0x02,0x01,0x00,0xff,0x00,0x10,0x12,0x12,0x12,0xff,0x12,0x12,0x12,0x10,0x00,0x00,},
/* 3 动 */ {0xe5,0x8a,0xa8,0x00,0x40,0x44,0xc4,0x44,0x44,0x44,0x40,0x10,0x10,0xff,0x10,0x10,0x10,0xf0,0x00,0x00,0x10,0x3c,0x13,0x10,0x14,0xb8,0x40,0x30,0x0e,0x01,0x40,0x80,0x40,0x3f,0x00,0x00,},
};
const Font font16x16 = {16, 16, (const uint8_t *)zh16x16, sizeof(zh16x16)/36, &afont16x8};
//这里改用sizeof来判断大小，注意要除36
```

* #### 图片显示
  * 先在取模助手取模，然后整个复制到`font.c`文件最后
  * 再将const的内容在.h文件中`extern`导出
  * 最后在主循环中`for循环`64次，循环调用`OLED_DrawImage`(0, 10, &jideImg, OLED_COLOR_NORMAL)
  * 此函数`第三个参数`填写图片名字的指针。

## 注意事项
* #### 在`OLED_Init()` 之前先`延迟`20ms

# 二、定时器
## 内部时钟定时器
* #### MX配置
  * 开启定时器，勾选内部时钟auto-reload preload，或者外部时钟ETR  
  * 若开启了外部时钟，还可以进行滤波、极性翻转、预分频功能  
  * 配置预分频器（Prescaler），例如：预分频值=7200-1  
  * 配置自动重装载寄存器（Counter Period）,例如：10000-1
  * 开启影子自动重装载寄存器（auto-reload preload）  
  * 打开NVIC中断  
* #### 程序一开始就产生一次更新中断
  * **原因**：`MX_TIM2_Init()`函数会将更新中断标志位 置1
  * **解决**：在定时器启动函数`HAL_TIM_Base_Start_IT(&htim2)`之前，用`__HAL_TIM_CLEAR_FLAG(&htim2,TIM_FLAG_UPDATE)`或者`__HAL_TIM_CLEAR_IT(&htim2,TIM_FLAG_UPDATE)`将此标志位清除
* #### 代码示例
```c
//开启定时器（带有中断）
HAL_TIM_Base_Start_IT(&htim2);

//更新中断回调函数,会自动清除更新中断标志位
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  if (htim == &htim4) {
    HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
  }
}

//读取计数寄存器
counter=__HAL_TIM_GET_COUNTER(&htim2);
```
## 外部时钟--ETR
* 芯片主频保持8MHz，或者调节预分频器降低TIM的内部时钟频率
  * 内部时钟会影响滤波器的作用，内部时钟8MHz最好
* 选择有外部时钟ETR的定时器，将`时钟源设置为ETR`
* 将**Clock Fliter**(滤波器)的值设为 1~15
* 计数寄存器会随着`上升沿`的到来计数
* 配合灰度传感器的DO引脚输出的波形，实现计数

## 定时器从模式（SlaveMode）-- 复位
* #### MX配置
  * 开启定时器后（内部时钟源），将从模式设置为`Reset Mode`
  * 将`Trigger Source`设置为TI1FP1（检测PA0引脚的上升沿）
  * 将PA0引脚设为`下拉`，避免浮空输入
  * 将**Clock Fliter**(滤波器)的值设为 1~15
* #### 在引脚接检测到上升沿信号后，`计数值置零`、`触发定时器更新中断`、`触发器中断标志位置1`
* #### 触发器中断标志位用函数`__HAL_TIM_GET_FLAG(htim,TIM_FLAG_TRIGGER)`查看
```c
//示例
 void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  if (htim->Instance == TIM2) {
    //判断是否位从模式触发中断
    if(__HAL_TIM_GET_FLAG(&htim2,TIM_FLAG_TRIGGER) == SET)
    {
      //需手动将TIM的触发器中断标志位 置0
      __HAL_TIM_CLEAR_FLAG(&htim2,TIM_FLAG_TRIGGER);
      HAL_UART_Transmit(&huart1, (uint8_t*)message2, strlen(message2), 10);
    }
    else
    {
       HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
       HAL_UART_Transmit(&huart1, (uint8_t*)message1, strlen(message1), 10);
    }
  }
}
```
## 定时器从模式（SlaveMode）-- 门模式
* #### 输入为高电平，开始计数；低电平时，停止计数
  * 高低电平可以设置边沿检测器取反
* #### 检测到上升沿或者下降沿都会将触发器中断标志位 置1
* #### 不复位计数器的值（不触发定时器更新中断，不会进入更新中断回调函数）
* #### MX配置
  * 开启定时器后（内部时钟源），将从模式设置为`Gated Mode`
  * 可在`Trigger Polarity`处实现高低电平取反
  * 可将PA0引脚设为`下拉`，避免浮空输入
  * 将`Clock Fliter`(滤波器)的值设为 1~15

## 定时器从模式（SlaveMode）-- 触发模式
* #### 在检测到设定的边沿类型后，开始计数，但*无法停止*
* #### 检测到上升下降沿都会将触发器中断标志位 置1
* #### 单脉冲模式
  * 一般配合此 One Pulse Mode 模式来使用
  * 计数到重装值就停止计数
  * 但再次检测到设定边沿后，会再次计数
* #### MX配置
  * 开启定时器后（内部时钟源），将从模式设置为`Trigger Mode`
  *  **勾选“One Pulse Mode”**
  * 可在`Trigger Polarity`选择要触发的边沿类型
  * 可将PA0引脚设为`下拉`，避免浮空输入
  * 将`Clock Fliter`(滤波器)的值设为 1~15

## 定时器输入捕获--超声波模块
* #### 输入捕获（测量脉冲宽度）
  * 将输入通道捕获到的上升沿/下降沿时
  * 立刻将计数器的值读取到**捕获寄存器**中（+开启中断）
  * 程序随时（一般在中断中）读取捕获寄存器的值
  * 注意：
    * 一个通道不能同时读取上升和下降沿，一般将通道12，通道34，结合起来用
    * 如通道1读取上升沿，通道2读取下降沿
    * 计数值一直在增加，只是利用输入捕获获取两个时刻的计数值  
    （使得超声波代码中最好先清零计数值，避免计数值达到自动重装值）
* #### MX配置
  * 将PA11设置为超声波的输入引脚Trig
  * 开启TIM1，选择内部时钟源
    * PA10为TIM1的输入捕获通道3
    * 所以开启`CHANNEL3`为直接输入模式
    * 开启`CHANNEL4`为间接输入模式
    * 将下方参数设置的`CHANNEL4`的极性为下降沿；`CHANNEL3`的极性为上升沿
    * 将下方NVIC设置中的捕获比较中断打开
  
* #### 输入捕获代码示例
```c
//时钟初始化、输入捕获通道初始化
HAL_TIM_Base_Start(&htim1);
HAL_TIM_IC_Start(&htim1 , TIM_CHANNEL_3);
HAL_TIM_IC_Start_IT(&htim1 , TIM_CHANNEL_4);//加上IT 开启中断
```
```c
//定义一下刚刚开启的中断
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
  //判断是否为定时器1 ，以及是否为通道4的输入捕获事件
  if(htim== &htim1 && htim->Channel == HAL_TIM_ACTIVE_CHANNEL_4)
  {
    upEdge = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_3);
    downEdge = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_4);
    distance = (float)(downEdge - upEdge) * 0.0343; 
    // 时钟频率为72MHz，分频为72-1，计算距离
  }
}
```
* #### 超声波代码示例
  * 超声波Trig引脚`拉高再拉低`，发送超声波
  * 超声波Trig拉低之后，Echo引脚`拉高`
  * 直到收到反弹的超声波，Echo引脚`拉低`
  * 通过测量Echo引脚`高低电平之间的时间`，来计算距离
  * Echo拉高时，**需手动将计数器清空**（从0开始计数）
```c
    //超声波模块
    HAL_GPIO_WritePin(Trig_GPIO_Port, Trig_Pin, GPIO_PIN_SET);
    HAL_Delay(1);
    HAL_GPIO_WritePin(Trig_GPIO_Port, Trig_Pin, GPIO_PIN_RESET);
    __HAL_TIM_SET_COUNTER(&htim1, 0); //清空计数器
    HAL_Delay(20);
```

# 三、PWM
* #### PWM：调节高低电平的占空比，实现对模拟信号的模拟
* #### 公式
  * $PWM周期 = \frac{(预分频器+1) \times (自动重装载值+1)}{时钟源频率}$

  * $占空比 = \frac{CCR}{自动重装载值} \times 100\%$

## 1.定时器输出比较模式
* #### 作用：将比较寄存器的值一直与计数器值进行比较
* #### 模式：冻结模式、强制模式、匹配模式、pwm模式12
* #### 在pwm模式下
  * 将计数值与比较寄存器的值进行比较
  * 两种大小关系对应着两种输出电平
  * 通过调节比较寄存器值得大小来调节占空比
  * 默认设置下（pwm1），比较寄存器值越大则占空比越大

## 2.MX配置
* #### 选择内部时钟源，设定预分频和自动重装载值
* #### 选择其中一个通道为PWM模式（PWM Geration CH1）
* #### 下方参数设置
  * 设置预分频值、自动重装载值（如：72-1，100-1）
  * `设置比较寄存器的值（Pulse）`
    * 有时候PWM输出引脚的小灯闪烁，不能保持稳定发光，可以通过赋一个较高的初始值解决(如：50)
  * 高速模式、输出极性选择

## 3.代码示例
```c
// 开启对应定时器和通道的PWM模式
  HAL_TIM_PWM_Start(&htim1,TIM_CHANNEL_1);
//设置比较寄存器的值
__HAL_TIM_SET_COMPARE(&htim1,TIM_CHANNEL_1,i);
//呼吸灯代码示例
for(int i = 0; i<100;i++)
    {
      __HAL_TIM_SET_COMPARE(&htim1,TIM_CHANNEL_1,i);
      HAL_Delay(10);
    }
    for(int i =99; i>=0 ;i--)
    {
      __HAL_TIM_SET_COMPARE(&htim1,TIM_CHANNEL_1,i);
      HAL_Delay(10);
    }
```

# 四、编码器 

## 1. 增量型编码器

* **工作原理**
    * 旋转时，两个输出引脚 A、B 分别输出两个方波
    * **顺时针转时**：A 方波落后 B 方波 90°
        * 例如：顺时针旋转时，读取 A 上升沿，B 为低电平
    * **逆时针转时**：A 方波超前 B 方波 90°
        * 例如：逆时针旋转时，读取 A 下降沿，B 为高电平
* **方向判断**
    * 读取 A、B 方波的电平状态，即可判断旋转方向
    * 可以通过 `GPIO_ReadPin()` 函数读取 A、B 的电平状态
    * **推荐方法**：将 A、B 分别接入 TI1FP、TI2FP 通道（最终接入编码器）
        * TI1FP、TI2FP 通道自带滤波器
        * 计数 A 通道的上升沿和下降沿（旋转一次计数 +2）

## 2.MX 配置

* **打开 TIM1**
    * 不需选择内部时钟源
    * 设置 `Combined Channel` 为 **Encoder Mode**
    * PA9、PA10 为 A、B 通道
    * `Encoder Mode` 选择其中一个通道进行计数
    * **设置预分频值**：`2-1`，使旋转一次计数 +1
    * 设置极性可以使正转和反转的计数方向不同（增、减）

## 3.代码示例
```c
// 开启编码器计数
 HAL_TIM_Encoder_Start(&htim1,TIM_CHANNEL_ALL);
 // 读取编码器计数值
 count = __HAL_TIM_GET_COUNTER(&htim1);
 //控制计数值在0~100之间
 if(count >6000){
      count = 0;
      __HAL_TIM_SET_COUNTER(&htim1,0);
    }
 else if (count >100){
      count = 100;
      __HAL_TIM_SET_COUNTER(&htim1,100);
    }
```  

