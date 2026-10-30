# Experiment: Design and Implementation of a Water Level Indicator Using STM32

## Aim

To design and implement a **Water Level Indicator using STM32 Nucleo-L031K6** that measures the water level using an analog input and displays the water level as **LOW, MEDIUM, or HIGH** on the Wokwi Serial Monitor.

---

## Components Required

- STM32 Nucleo-L031K6
- Potentiometer to simulate the water-level sensor
- Wokwi Simulator
- Virtual Serial Monitor
- Connecting wires

---

## Theory

A **Water Level Indicator** is used to monitor the amount of water present in a tank.

In this experiment, a **potentiometer** is used in Wokwi to simulate the output of a water-level sensor. The potentiometer produces an analog voltage between **0 V and 3.3 V**.

The analog signal is connected to the **PA0 ADC input** of the STM32 Nucleo-L031K6.

The STM32 contains a **12-bit Analog-to-Digital Converter (ADC)**. Therefore, the analog input voltage is converted into a digital value between:

- **0** → Minimum water level
- **4095** → Maximum water level

The ADC value is converted into a percentage, and the water level is classified as:

- **LOW**
- **MEDIUM**
- **HIGH**

The water-level information is displayed on the **Wokwi Serial Monitor** using UART communication.

---

## Pin Configuration

| Component | STM32 Pin | Function |
|---|---|---|
| Potentiometer SIG | PA0 | ADC input |
| Potentiometer VCC | 3.3V | Power supply |
| Potentiometer GND | GND | Ground |
| USART2 TX | PA2 | Serial data transmission |
| USART2 RX | PA15 | Serial data reception |

---

## Block Diagram

~~~text
       Potentiometer
   (Water Level Sensor)
          │
          │ Analog Signal
          ▼
       PA0 / ADC
          │
          ▼
+-----------------------+
| STM32 Nucleo-L031K6   |
|                       |
| ADC Conversion        |
|        ↓              |
| Water Level %         |
|        ↓              |
| LOW / MEDIUM / HIGH   |
+-----------+-----------+
            │
            │ USART2
            ▼
      Serial Monitor
            │
            ▼
   Water Level Display
~~~

---

## Water Level Classification

| Water Level | Percentage | Indication |
|---|---:|---|
| Low | 0–30% | LOW |
| Medium | 31–70% | MEDIUM |
| High | 71–100% | HIGH |

---

## Algorithm

1. Start the program.
2. Initialize the STM32 HAL library.
3. Configure the system clock.
4. Configure **PA0 as an analog input**.
5. Initialize ADC1.
6. Initialize USART2 for serial communication.
7. Read the analog value from the potentiometer.
8. Convert the ADC reading into water-level percentage.
9. Compare the percentage with predefined limits.
10. If the water level is less than or equal to 30%, display **LOW**.
11. If the water level is between 31% and 70%, display **MEDIUM**.
12. If the water level is greater than 70%, display **HIGH**.
13. Display the ADC value, water-level percentage, and status on the Serial Monitor.
14. Wait for one second.
15. Repeat continuously.

---

## Program

```c
#include "main.h"
#include <stdio.h>
#include <string.h>

ADC_HandleTypeDef hadc;
UART_HandleTypeDef huart2;

uint32_t adc_value;
uint32_t water_level;
char message[100];

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC_Init(void);
static void MX_USART2_UART_Init(void);

int main(void)
{
    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();
    MX_ADC_Init();
    MX_USART2_UART_Init();

    while (1)
    {
        HAL_ADC_Start(&hadc);

        HAL_ADC_PollForConversion(&hadc, HAL_MAX_DELAY);

        adc_value = HAL_ADC_GetValue(&hadc);

        HAL_ADC_Stop(&hadc);

        water_level = (adc_value * 100) / 4095;

        if (water_level <= 30)
        {
            sprintf(message,
                    "ADC Value: %lu\r\nWater Level: %lu%%\r\nStatus: LOW\r\n\r\n",
                    adc_value, water_level);
        }
        else if (water_level <= 70)
        {
            sprintf(message,
                    "ADC Value: %lu\r\nWater Level: %lu%%\r\nStatus: MEDIUM\r\n\r\n",
                    adc_value, water_level);
        }
        else
        {
            sprintf(message,
                    "ADC Value: %lu\r\nWater Level: %lu%%\r\nStatus: HIGH\r\n\r\n",
                    adc_value, water_level);
        }

        HAL_UART_Transmit(&huart2,
                          (uint8_t *)message,
                          strlen(message),
                          HAL_MAX_DELAY);

        HAL_Delay(1000);
    }
}

void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_MSI;
    RCC_OscInitStruct.MSIState = RCC_MSI_ON;
    RCC_OscInitStruct.MSICalibrationValue = 0;
    RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_5;
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;

    HAL_RCC_OscConfig(&RCC_OscInitStruct);

    RCC_ClkInitStruct.ClockType =
        RCC_CLOCKTYPE_HCLK |
        RCC_CLOCKTYPE_SYSCLK |
        RCC_CLOCKTYPE_PCLK1;

    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_MSI;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;

    HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0);
}

static void MX_ADC_Init(void)
{
    ADC_ChannelConfTypeDef sConfig = {0};

    hadc.Instance = ADC1;
    hadc.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV1;
    hadc.Init.Resolution = ADC_RESOLUTION_12B;
    hadc.Init.DataAlign = ADC_DATAALIGN_RIGHT;
    hadc.Init.ScanConvMode = ADC_SCAN_DIRECTION_FORWARD;
    hadc.Init.EOCSelection = ADC_EOC_SINGLE_CONV;
    hadc.Init.LowPowerAutoWait = DISABLE;
    hadc.Init.LowPowerAutoPowerOff = DISABLE;
    hadc.Init.ContinuousConvMode = DISABLE;
    hadc.Init.DiscontinuousConvMode = DISABLE;
    hadc.Init.ExternalTrigConv = ADC_SOFTWARE_START;
    hadc.Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
    hadc.Init.DMAContinuousRequests = DISABLE;
    hadc.Init.Overrun = ADC_OVR_DATA_OVERWRITTEN;

    HAL_ADC_Init(&hadc);

    sConfig.Channel = ADC_CHANNEL_0;
    sConfig.Rank = ADC_RANK_CHANNEL_NUMBER;
    sConfig.SamplingTime = ADC_SAMPLETIME_39CYCLES_5;

    HAL_ADC_ConfigChannel(&hadc, &sConfig);
}

static void MX_USART2_UART_Init(void)
{
    huart2.Instance = USART2;
    huart2.Init.BaudRate = 115200;
    huart2.Init.WordLength = UART_WORDLENGTH_8B;
    huart2.Init.StopBits = UART_STOPBITS_1;
    huart2.Init.Parity = UART_PARITY_NONE;
    huart2.Init.Mode = UART_MODE_TX_RX;
    huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
    huart2.Init.OverSampling = UART_OVERSAMPLING_16;

    HAL_UART_Init(&huart2);
}

static void MX_GPIO_Init(void)
{
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    __HAL_RCC_GPIOA_CLK_ENABLE();

    GPIO_InitStruct.Pin = GPIO_PIN_0;
    GPIO_InitStruct.Mode = GPIO_MODE_ANALOG;
    GPIO_InitStruct.Pull = GPIO_NOPULL;

    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}
```

---

## Circuit Connections

### Potentiometer

| Potentiometer Pin | STM32 Connection |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SIG / Middle Pin | PA0 |

---

## Circuit Diagram

~~~text
              Potentiometer
         (Water Level Sensor)
        +--------------------+
3.3V ---| VCC                |
GND  ---| GND                |
        | SIG                 |
        +--+-----------------+
           |
           |
          PA0
           |
           v
+---------------------------+
| STM32 Nucleo-L031K6       |
|                           |
| PA0  -> ADC Input         |
|                           |
| PA2  -> USART2 TX         |
| PA15 -> USART2 RX         |
+-------------+-------------+
              |
              | UART
              v
       Wokwi Serial Monitor
~~~

---

## Procedure

1. Open **Wokwi**.
2. Select the **STM32 Nucleo-L031K6** board.
3. Add a **potentiometer**.
4. Connect the potentiometer VCC pin to **3.3V**.
5. Connect the potentiometer GND pin to **GND**.
6. Connect the potentiometer SIG pin to **PA0**.
7. Enter the STM32 HAL program.
8. Compile the program.
9. Start the simulation.
10. Open the **Serial Monitor**.
11. Rotate the potentiometer to simulate different water levels.
12. Observe the ADC value, water-level percentage, and water-level status.

---

## Expected Output

### Low Water Level

~~~text
ADC Value: 800
Water Level: 19%
Status: LOW
~~~

### Medium Water Level

~~~text
ADC Value: 2200
Water Level: 53%
Status: MEDIUM
~~~

### High Water Level

~~~text
ADC Value: 3500
Water Level: 85%
Status: HIGH
~~~

---

## Working

The potentiometer produces an analog voltage that represents the water level in the tank.

The STM32 reads this analog voltage through the **PA0 ADC input**. Since the ADC has a **12-bit resolution**, the analog signal is converted into a digital value between **0 and 4095**.

The ADC value is converted into water-level percentage using:

`Water Level (%) = (ADC Value × 100) / 4095`

The STM32 then classifies the water level as:

- **0–30% → LOW**
- **31–70% → MEDIUM**
- **71–100% → HIGH**

The ADC value, percentage, and water-level status are transmitted through **USART2** and displayed on the **Wokwi Serial Monitor**.

---

## Result

Thus, the **Water Level Indicator using STM32 Nucleo-L031K6** was designed and implemented successfully. The water level was measured using the ADC and displayed as **LOW, MEDIUM, or HIGH** on the Wokwi Serial Monitor.
