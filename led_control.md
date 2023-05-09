<script>
    const h1 = document.querySelector(`h1`)
    const a = h1.querySelector(`a`)
    if (a.href===`https://mark.yrf.wiki/`) h1.style.display = 'none'
</script>

```c
/* USER CODE BEGIN Header */
/* USER CODE END Header */

/* Includes ------------------------------------------------------------------*/
#include "main.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include "string.h"
#include "stdio.h"
#include "los_sys.h"
#include "los_typedef.h"
#include "los_task.ph"
#include "los_queue.h"
#include "los_sem.h"
/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */
typedef enum
{
    LED_ON = 0,
    LED_OFF
} LedStatus;

typedef enum
{
    LED0 = 0,
    LED1,
    LED2,
    LED3,
    LED4
} LedIndex;
typedef struct
{
    int reverse;
    int delayms;
} LedConfig;
typedef struct
{
    GPIO_TypeDef *port;
    uint16_t pin;
} LedPin;
/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
#define LED_PIN(port, pin) \
    {                      \
        port, pin          \
    }

#define NUM_LEDS 5
#define LED_INITIAL_STATE LED_OFF
/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/
UART_HandleTypeDef huart1;

/* USER CODE BEGIN PV */
unsigned char UART1_Rx_Buf[255] = {0};
unsigned char UART1_Rx_flag = 0;
unsigned char UART1_Rx_cnt = 0;
unsigned char UART1_Rx_temp[1] = {0};
unsigned char arrindex = 0, reverse = 0, isKeyScan = 0;
unsigned int delayms = 300;
UINT32 Key_Task_Handle;
UINT32 Test2_Task_Handle;
/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_USART1_UART_Init(void);
/* USER CODE BEGIN PFP */
static UINT32 Creat_Test2_Task(void);
static UINT32 Creat_Key_Task();
static void Test2_Task(void);
unsigned char key_scan(void);
/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
LedPin ledPins[NUM_LEDS] = {
    LED_PIN(LED0_GPIO_Port, LED0_Pin),
    LED_PIN(LED1_GPIO_Port, LED1_Pin),
    LED_PIN(LED2_GPIO_Port, LED2_Pin),
    LED_PIN(LED3_GPIO_Port, LED3_Pin),
    LED_PIN(LED4_GPIO_Port, LED4_Pin)};
LedStatus ledStates[NUM_LEDS] = {
    LED_INITIAL_STATE,
    LED_INITIAL_STATE,
    LED_INITIAL_STATE,
    LED_INITIAL_STATE,
    LED_INITIAL_STATE};
void UART_SendByte(USART_TypeDef *Uart, uint8_t data)
{
    Uart->DR = data;
    while ((Uart->SR & UART_FLAG_TXE) == 0)
        ;
    while ((Uart->SR & UART_FLAG_TC) == 0)
        ;
}
#pragma import(__use_no_semihosting)

struct __FILE
{
    int handle;
};

FILE __stdout;

void _sys_exit(int x)
{
    x = x;
}
int fputc(int ch, FILE *f)
{

    UART_SendByte(USART1, (uint8_t)ch);
    return (ch);
}
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if (huart->Instance == USART1)
    {
        UART1_Rx_Buf[UART1_Rx_cnt++] = UART1_Rx_temp[0];
        if (0x0a == UART1_Rx_temp[0])
            UART1_Rx_flag = 1;
    }
    HAL_UART_Receive_IT(&huart1, UART1_Rx_temp, 1);
}
void write_led_states()
{
    for (LedIndex i = 0; i < NUM_LEDS; i++)
    {
        if (ledPins[i].port != NULL)
        {
            uint16_t pin_state = (ledStates[i] == LED_ON) ? GPIO_PIN_RESET : GPIO_PIN_SET;
            HAL_GPIO_WritePin(ledPins[i].port, ledPins[i].pin, pin_state);
        }
    }
}
void led_task()
{
    while (!isKeyScan)
    {
        LOS_TaskDelay(10);
    }

    ledStates[LED0] = LED_OFF;
    ledStates[LED1] = LED_OFF;
    ledStates[LED2] = LED_OFF;
    ledStates[LED3] = LED_OFF;
    ledStates[LED4] = LED_OFF;

    LedIndex nextIndex = (reverse == 0) ? (arrindex + 1) % NUM_LEDS : (arrindex + NUM_LEDS - 1) % NUM_LEDS;

    ledStates[arrindex] = LED_ON;
    write_led_states();

    printf("当前亮的灯为：LED%d\r\n", arrindex);

    arrindex = nextIndex;

    LOS_TaskDelay(delayms);
}
void led_config(LedConfig config)
{
    arrindex = (config.reverse == 0) ? 0 : NUM_LEDS - 1;
    for (LedIndex i = 0; i < NUM_LEDS; i++)
    {
        ledStates[i] = LED_INITIAL_STATE;
    }
    write_led_states();

    reverse = config.reverse;
    delayms = config.delayms;
}
/* USER CODE END 0 */

/**
 * @brief  The application entry point.
 * @retval int
 */
int main(void)
{
    /* USER CODE BEGIN 1 */
    UINT32 uwRet = LOS_OK; // 定义一个任务创建的返回值，默认为创建成功
    /* USER CODE END 1 */

    /* MCU Configuration--------------------------------------------------------*/

    /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
    HAL_Init();

    /* USER CODE BEGIN Init */

    /* USER CODE END Init */

    /* Configure the system clock */
    SystemClock_Config();

    /* USER CODE BEGIN SysInit */

    /* USER CODE END SysInit */

    /* Initialize all configured peripherals */
    MX_GPIO_Init();
    MX_USART1_UART_Init();
    /* USER CODE BEGIN 2 */
    uwRet = LOS_KernelInit();
    if (uwRet != LOS_OK)
    {
        printf("LiteOS 核心初始化失败！\n");
        return LOS_NOK;
    }
    uwRet = Creat_Test2_Task();
    if (uwRet != LOS_OK)
    {
        printf("Test2_Task任务创建失败！\n");
        return LOS_NOK;
    }
    uwRet = Creat_Key_Task();
    if (uwRet != LOS_OK)
    {
        printf("Key_Task任务创建失败！\n");
        return LOS_NOK;
    }
    led_config((LedConfig){.reverse = 0, .delayms = 300});
    LOS_Start();

    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
    }
    /* USER CODE END 3 */
}

/**
 * @brief System Clock Configuration
 * @retval None
 */
void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    /** Initializes the CPU, AHB and APB busses clocks
     */
    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
    RCC_OscInitStruct.HSEState = RCC_HSE_ON;
    RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
    RCC_OscInitStruct.HSIState = RCC_HSI_ON;
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
    RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
    if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
    {
        Error_Handler();
    }
    /** Initializes the CPU, AHB and APB busses clocks
     */
    RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
    RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

    if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
    {
        Error_Handler();
    }
}

/**
 * @brief USART1 Initialization Function
 * @param None
 * @retval None
 */
static void MX_USART1_UART_Init(void)
{

    /* USER CODE BEGIN USART1_Init 0 */

    /* USER CODE END USART1_Init 0 */

    /* USER CODE BEGIN USART1_Init 1 */

    /* USER CODE END USART1_Init 1 */
    huart1.Instance = USART1;
    huart1.Init.BaudRate = 115200;
    huart1.Init.WordLength = UART_WORDLENGTH_8B;
    huart1.Init.StopBits = UART_STOPBITS_1;
    huart1.Init.Parity = UART_PARITY_NONE;
    huart1.Init.Mode = UART_MODE_TX_RX;
    huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
    huart1.Init.OverSampling = UART_OVERSAMPLING_16;
    if (HAL_UART_Init(&huart1) != HAL_OK)
    {
        Error_Handler();
    }
    /* USER CODE BEGIN USART1_Init 2 */
    HAL_UART_Receive_IT(&huart1, UART1_Rx_temp, 1);
    /* USER CODE END USART1_Init 2 */
}

/**
 * @brief GPIO Initialization Function
 * @param None
 * @retval None
 */
static void MX_GPIO_Init(void)
{
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    /* GPIO Ports Clock Enable */
    __HAL_RCC_GPIOC_CLK_ENABLE();
    __HAL_RCC_GPIOD_CLK_ENABLE();
    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOB_CLK_ENABLE();

    /*Configure GPIO pin Output Level */
    HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_RESET);

    /*Configure GPIO pin Output Level */
    HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_RESET);

    /*Configure GPIO pin Output Level */
    HAL_GPIO_WritePin(GPIOB, LED2_Pin | LED3_Pin | LED4_Pin, GPIO_PIN_RESET);

    /*Configure GPIO pin : KEY2_Pin */
    GPIO_InitStruct.Pin = KEY2_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    HAL_GPIO_Init(KEY2_GPIO_Port, &GPIO_InitStruct);

    /*Configure GPIO pin : KEY0_Pin */
    GPIO_InitStruct.Pin = KEY0_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
    GPIO_InitStruct.Pull = GPIO_PULLUP;
    HAL_GPIO_Init(KEY0_GPIO_Port, &GPIO_InitStruct);

    /*Configure GPIO pin : LED0_Pin */
    GPIO_InitStruct.Pin = LED0_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(LED0_GPIO_Port, &GPIO_InitStruct);

    /*Configure GPIO pin : KEY1_Pin */
    GPIO_InitStruct.Pin = KEY1_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
    GPIO_InitStruct.Pull = GPIO_PULLUP;
    HAL_GPIO_Init(KEY1_GPIO_Port, &GPIO_InitStruct);

    /*Configure GPIO pin : LED1_Pin */
    GPIO_InitStruct.Pin = LED1_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(LED1_GPIO_Port, &GPIO_InitStruct);

    /*Configure GPIO pins : LED2_Pin LED3_Pin LED4_Pin */
    GPIO_InitStruct.Pin = LED2_Pin | LED3_Pin | LED4_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);
}

/* USER CODE BEGIN 4 */
static void Test2_Task(void)
{
    while (1)
    {
        led_task();
    }
}
static void Key_Task(void)
{
	  unsigned char key_value;
    printf("等待按键操作\r\n");
    while (1)
    {
        key_value = key_scan();
        switch (key_value)
        {
        case 1:
            led_config((LedConfig){.reverse = 0, .delayms = 300});
						isKeyScan = 1;
            break;
        case 2:
            led_config((LedConfig){.reverse = 1, .delayms = 300});
						isKeyScan = 1;
            break;
        case 3:
            led_config((LedConfig){.reverse = 0, .delayms = 100});
						isKeyScan = 1;
            break;
        default:
            break;
        }
        LOS_TaskDelay(10);
    }
}
static UINT32 Creat_Test2_Task()
{
    UINT32 uwRet = LOS_OK; /* 定义一个创建任务的返回类型，初始化为创建成功的返回值 */
    TSK_INIT_PARAM_S task_init_param;
    task_init_param.usTaskPrio = 4;                               /* 优先级，数值越小，优先级越高 */
    task_init_param.pcName = "Test2_Task";                        /* 任务名，字符串形式，方便调试 */
    task_init_param.pfnTaskEntry = (TSK_ENTRY_FUNC)Test2_Task;    /* 任务函数名 */
    task_init_param.uwStackSize = 0x0150;                         /* 栈大小，单位为字，即4个字节 */
    uwRet = LOS_TaskCreate(&Test2_Task_Handle, &task_init_param); /* 创建任务 */
    return uwRet;
}
static UINT32 Creat_Key_Task()
{
    UINT32 uwRet = LOS_OK; /* 定义一个创建任务的返回类型，初始化为创建成功的返回值 */
    TSK_INIT_PARAM_S task_init_param;
    task_init_param.usTaskPrio = 7;                             /* 优先级，数值越小，优先级越高 */
    task_init_param.pcName = "Key_Task";                        /* 任务名，字符串形式，方便调试 */
    task_init_param.pfnTaskEntry = (TSK_ENTRY_FUNC)Key_Task;    /* 任务函数名 */
    task_init_param.uwStackSize = 0x0150;                       /* 栈大小，单位为字，即4个字节 */
    uwRet = LOS_TaskCreate(&Key_Task_Handle, &task_init_param); /* 创建任务 */
    return uwRet;
}
unsigned char key_scan(void)
{
    if (HAL_GPIO_ReadPin(KEY0_GPIO_Port, KEY0_Pin) == 0)
        return 1;
    else if (HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin) == 0)
        return 2;
    else if (HAL_GPIO_ReadPin(KEY2_GPIO_Port, KEY2_Pin) == 0)
        return 3;
    else
        return 0;
}
/* USER CODE END 4 */

/**
 * @brief  This function is executed in case of error occurrence.
 * @retval None
 */
void Error_Handler(void)
{
    /* USER CODE BEGIN Error_Handler_Debug */
    /* User can add his own implementation to report the HAL error return state */

    /* USER CODE END Error_Handler_Debug */
}

#ifdef USE_FULL_ASSERT
/**
 * @brief  Reports the name of the source file and the source line number
 *         where the assert_param error has occurred.
 * @param  file: pointer to the source file name
 * @param  line: assert_param error line source number
 * @retval None
 */
void assert_failed(uint8_t *file, uint32_t line)
{
    /* USER CODE BEGIN 6 */
    /* User can add his own implementation to report the file name and line number,
       tex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
    /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

/************************ (C) COPYRIGHT STMicroelectronics *****END OF FILE****/

```
