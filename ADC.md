#硬件 #STM32 #片上外设

> *ADC（Analog-Digital Converter）模拟-数字转换器*
> 
> 可以将引脚上连续变化的模拟电压转换为内存中存储的数字变量，
> 建立模拟电路到数字电路的桥梁。

![[ADC0809原理图.png]]
## 原理

将输入电压与内部参考电压（VREF）比较，并使用二分法逐次逼近，最终得到测量电压值。

>简而言之就是一个比较器。

由于二分法，用二进制来看，在比较过程中，实际就是就是在判断从前往后每一位是0还是1。

ADC的重要参数是`位数`，即输出结果的二进制位数。多少位即代表二分多少次。

## 在 #STM32 中：

### 工作频率

ADC的时钟（即[[ADCCLK]]）来自于[[RCC]]的[[ADC预分频器]]，其主频为**72MHz**，支持2、4、6、8级分频。

[[ADCCLK]]最高支持**14MHz**，因此在[[ADC预分频器]]应应设置6分频（12MHz）和8分频（9MHZ）。

![[STM32中的ADC基本结构图.png]]

### 规则组的转换方式

>即单次/连续转换与扫描/非扫描模式组合出来的四种模式。

#### 单次/连续转换:
##### 单次转换
##### 连续转换
#### 扫描/非扫描模式:
##### 扫描模式
##### 非扫描模式

### 触发控制

>看不懂

![[Pasted image 20251107001445.png]]

### 数据对齐模式

STM32的[[ADC]]为12位[[ADC]]，但数据寄存器为12位，因此需要设置数据对齐模式。

>一般都是使用的右对齐模式，这样输出值不用转换可以直接使用。
#### 右对齐模式

![[Pasted image 20251107001901.png]]
#### 左对齐模式

![[Pasted image 20251107001924.png]]

### 转换时间

>在使用ADC采集电压值的时候，外部电压可能会变化，因此ADC内部会存在一个电容以存储待采样的电压，这个电容由一个开关所控制，以便在每次采样完成后更新电容内的电压。*那么这个开关开闭的频率应该是多少呢*？
>
>这就是ADC的**转换时间**。

STM32 ADC的总转换时间为：

```
TCONV = 采样时间 + 12.5个ADC周期
```

> [!example]
> 当ADCCLK=14MHz，采样时间为1.5个ADC周期，则：
>```
>TCONV = 1.5 + 12.5 = 14个ADC周期 = 1μs
>```

### 校准

ADC有一个内置自校准模式。校准可大幅减小因内部电容器组的变化而造成的准精度误差。校准期间，在每个电容器上都会计算出一个误差修正码(数字值)，修正码可用于消除在随后的转换中每个电容器上产生的误差。

> [!info]
> 建议在每次上电后执行一次校准。

启动校准前， ADC必须处于关电状态超过至少两个ADC时钟周期。

>然而事实上关于校准，我们只需要在初始化ADC后加几行校准代码就行了，没必要研究那么深。

### 硬件电路

| 采集电位器的阻值               | 采集传感器（即可变电阻）值     | 采集高电压值          |
| ---------------------- | ----------------- | --------------- |
| ![[采集传感器（即可变电阻）值.png]] | ![[采集电位器的阻值.png]] | ![[采集高电压值.png]] |

>不过不推荐使用上述采集高电压值的方法，更推荐使用隔离放大器等设备，以实现高低压分离。

### 在 #HAL库 中使用ADC


>[!info]
>这里主要讲一下*单次非扫描模式*

##### 1. 配置ADC参数

1. 选择通道为IN0；
2. 选择为独立模式；（对应标准库中的 `ADC_InitStruct.ADC_Mode=ADC_Mode_Independent`）
3. 设置数据为右对齐、非扫描模式、非连续模式。；
4. 设置转换通道数为1，因为前面只勾选上了IN0。

![[Pasted image 20251107150123.png]]

#### 2. 配置ADC时钟

> [!quote]
> ADC的时钟（即[[ADCCLK]]）来自于[[RCC]]的[[ADC预分频器]]，其主频为**72MHz**，支持2、4、6、8级分频。
> 
>[[ADCCLK]]最高支持**14MHz**，因此在[[ADC预分频器]]应应设置6分频（12MHz）和8分频（9MHZ）。

在`Clock Configuration`中将[[ADC预分频器]]设置为至少六分频。

![[Pasted image 20251107150652.png]]

#### 3. 在main函数中校正ADC

```c
HAL_ADCEx_Calibration_Start(&hadc1);
```

#### 4. 编写转换并获取ADC函数

```C
/**  
 * @brief  获取ADC的转换值  
 * @param  hadc: 指向ADC句柄的指针  
 * @retval uint16_t: ADC转换的结果值。如果转换失败，返回0。  
 * * 此函数启动ADC转换，并等待转换完成。如果转换成功，  
 * 则返回转换结果；否则返回0。  
 */uint16_t ADC_Get_Value(ADC_HandleTypeDef *hadc) {  
    HAL_StatusTypeDef HalState;  
    uint16_t Ret;  
    HAL_ADC_Start(hadc);  
    HalState= HAL_ADC_PollForConversion(hadc, 10);  
    if(HalState == HAL_OK) {  
        Ret=HAL_ADC_GetValue(hadc);  
    } else {  
        Ret=0;  
    }    return Ret;  
}
```

>这里第一行使用了[[HAL_StatusTypeDef]]。

#### 5. while中获取显示

```C
uint16_t ADValue;
float Voltage;
int main(void) {
	HAL_Init();
	SystemClock_Config();
	MX_GPIO_Init();
	MX_ADC1_Init();
	OLED_Init();
	OLED_Clear();
	HAL_ADCEx_Calibration_Start(&hadc1);
	OLED_ShowString(1,1,"ADValue:");
	OLED_ShowString(2,1,"Voltage:0.00V")
	while (1) {
		ADValue= StartAndGetOneResult();
		OLED_ShowNum(1,9,ADValue,4);
		Voltage=(float) ADValue/ 4095 *3.3;
		OLED_ShowNum(2,9,(uint32_t)Voltage,1);
		OLED_ShowNum(2,11,((uint16_t)(Voltage * 100)) % 100,2);
		HAL_Delay(100);
	}
}
```

>[!info]
>这里需要注意的是`(uint32_t)Voltage`需要强制类型转换一下，否则会得到一个很长的数。














































### 其他

ADC1和ADC2可以配合工作，即双ADC模式，可以实现更高精度/更高速度的采样。

具体引脚定义：
![[STM32F103C8T6引脚定义.png]]