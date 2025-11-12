#STM32 #HAL库 

```c
//stm32f1xx_hal_def.h

/**
* @brief HAL Status structures definition
*/
typedef enum
{
	HAL_OK = 0x00U,
	HAL_ERROR = 0x01U,
	HAL_BUSY = 0x02U,
	HAL_TIMEOUT = 0x03U
} HAL_StatusTypeDef;
```

在写驱动代码的时候经常将函数类型指定为`HAL_StatusTypeDef`。