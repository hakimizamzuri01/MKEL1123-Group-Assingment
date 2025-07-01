# MKEL1123-Group1-Assingment milestone 1


Bluepill stm32 blinking project.

---

## 🚀 Getting Started

### Prerequisites

* `STM32CubeIDE`

### Step1: Start a project

Provide step-by-step instructions on how to install or set up your project.

1.  **Create a new project:Choose you specific STM32 model**
   ![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/c1.png?raw=true)



2.  **Chose the pin 13 as output:**

   
   ![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/c2.png?raw=true)


   
### Step2: Bare Metal Coding STM32

3.  **bare metal code for the STM32**
   This use the PIN 13 inside the STM32 as an output using 'ODR' and using 'HAL_WAIT' for delay for 1s, This will loop 1s ON and 1s OFF

   ![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/c3.png?raw=true)


[Full code for the assingment](main_code.c)
### Step3: Using SVlinkV2 to link to the hardware

1.  **Connection between the STM32 and SVLinkv2:**

  SVlink v2 pin layout
  
![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/q1.jpg?raw=true)

 make sure stm32 in the program mode
 
![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/q2.jpg?raw=true)  
   
2.  **upload the code in the STM32:**

The Green Icon on the top right is use for uploading the code to the STM32
![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/q4.png?raw=true)


### Step4: SET the STM32 in operating mode
![Alt text for the image](https://github.com/hakimizamzuri01/MKEL1123-Group-Assingment/blob/main/q3.jpg?raw=true)

---

### Google Video link
[Gdrive video](https://drive.google.com/file/d/1-Td6JxVvGqeYaTiAVWJ-KXsWRnB_5VQO/view?usp=drivesdk)


### Milestone 5 code
 ```c
#include "main.h"
#include "stm32f4xx.h"

// States
typedef enum {
    IDLE,
    NORMAL_ENTRY,
    NORMAL_GREEN,
    NORMAL_YELLOW,
    MAIN_GREEN,
    NORMAL_WAIT,
    MAIN_WAIT,
    WAIT_CHECK
} TrafficState;

TrafficState state = IDLE;
uint32_t timer = 0;
uint32_t loop_timer = 0;
uint8_t prev_normal = 0, prev_main = 0;

// === INPUT FUNCTIONS ===
uint8_t read_normal(void) {
    return ((GPIOB->IDR & (1 << 0)) == 0);  // B0 shared for normal switches
}

uint8_t read_main(void) {
    return ((GPIOA->IDR & (1 << 0)) == 0); // A0 shared for main switches
}

// === OUTPUT FUNCTIONS ===
// Normal LEDs: shared: B12 (RED), B13 (YELLOW), B14 (GREEN)
void set_normal(uint8_t red, uint8_t yellow, uint8_t green) {
    GPIOB->BSRR = ((red ? (1 << 12) : (1 << (12 + 16))) |
                   (yellow ? (1 << 13) : (1 << (13 + 16))) |
                   (green ? (1 << 14) : (1 << (14 + 16))));
}

// Main LEDs: shared: B8 (RED), B9 (YELLOW), B7 (GREEN)
void set_main(uint8_t red, uint8_t yellow, uint8_t green) {
    GPIOB->BSRR = ((red ? (1 << 8) : (1 << (8 + 16))) |
                   (yellow ? (1 << 9) : (1 << (9 + 16))) |
                   (green ? (1 << 7) : (1 << (7 + 16))));
}

void SystemClock_Config(void);

int main(void)
{
    HAL_Init();
    SystemClock_Config();

    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN | RCC_AHB1ENR_GPIOBEN;

    // OUTPUT Pins
    GPIOB->MODER |= (1 << (12 * 2)) | (1 << (13 * 2)) | (1 << (14 * 2)) | // Normal
                    (1 << (7 * 2))  | (1 << (8 * 2))  | (1 << (9 * 2));   // Main

    // INPUT Pins
    GPIOB->MODER &= ~(3 << (0 * 2)); // B0 input
    GPIOB->PUPDR |= (1 << (0 * 2));

    GPIOA->MODER &= ~(3 << (0 * 2)); // A0 input
    GPIOA->PUPDR |= (1 << (0 * 2));

    set_main(0, 0, 1);
    set_normal(1, 0, 0);

    while (1) {
        uint8_t normal = read_normal();
        uint8_t main = read_main();

        switch (state) {
            case IDLE:
                set_main(0, 0, 1);
                set_normal(1, 0, 0);

                if (normal && !prev_normal && !main) {
                    HAL_Delay(1000);
                    state = NORMAL_ENTRY;
                } else if (main && normal && (!prev_main || !prev_normal)) {
                    loop_timer = HAL_GetTick() + 10000;
                    state = MAIN_GREEN;
                }
                break;

            case NORMAL_ENTRY:
                set_main(0, 1, 0);
                set_normal(1, 0, 0);
                HAL_Delay(1000);

                set_main(1, 0, 0);
                set_normal(1, 0, 0);
                HAL_Delay(1000);

                set_normal(0, 0, 1);
                state = NORMAL_GREEN;
                break;

            case NORMAL_GREEN:
                if (main && !prev_main) {
                    HAL_Delay(1000);
                    set_normal(0, 1, 0);
                    set_main(1, 0, 0);
                    HAL_Delay(1000);

                    set_normal(1, 0, 0);
                    set_main(1, 0, 0);
                    HAL_Delay(1000);

                    set_main(0, 0, 1);
                    loop_timer = HAL_GetTick() + 10000;
                    state = MAIN_GREEN;
                } else if (!normal && prev_normal && !main) {
                    timer = HAL_GetTick() + 3000;
                    state = NORMAL_WAIT;
                } else if (main && normal && (!prev_main || !prev_normal)) {
                    loop_timer = HAL_GetTick() + 10000;
                    state = MAIN_GREEN;
                }
                break;

            case NORMAL_WAIT:
                if (read_normal()) {
                    set_normal(0, 0, 1);
                    state = NORMAL_GREEN;
                } else if (HAL_GetTick() >= timer) {
                    set_normal(0, 1, 0);
                    HAL_Delay(1000);

                    set_normal(1, 0, 0);
                    HAL_Delay(1000);

                    set_main(0, 0, 1);
                    state = IDLE;
                }
                break;

            case MAIN_GREEN:
                set_main(0, 0, 1);
                set_normal(1, 0, 0);

                if (HAL_GetTick() >= loop_timer) {
                    set_main(0, 1, 0);
                    set_normal(1, 0, 0);
                    HAL_Delay(1000);

                    set_main(1, 0, 0);
                    set_normal(1, 0, 0);
                    HAL_Delay(1000);

                    set_normal(0, 0, 1);
                    loop_timer = HAL_GetTick() + 5000;
                    state = NORMAL_YELLOW;
                } else if (!main && normal) {
                    timer = HAL_GetTick() + 3000;
                    state = MAIN_WAIT;
                }
                break;

            case MAIN_WAIT:
                if (read_main()) {
                    set_main(0, 0, 1);
                    state = MAIN_GREEN;
                } else if (HAL_GetTick() >= timer) {
                    set_main(0, 1, 0);
                    HAL_Delay(1000);

                    set_main(1, 0, 0);
                    HAL_Delay(1000);

                    set_normal(0, 0, 1);
                    state = NORMAL_GREEN;
                }
                break;

            case NORMAL_YELLOW:
                set_main(1, 0, 0);
                set_normal(0, 0, 1);

                if (HAL_GetTick() >= loop_timer) {
                    set_normal(0, 1, 0);
                    HAL_Delay(1000);

                    set_normal(1, 0, 0);
                    HAL_Delay(1000);

                    set_main(0, 0, 1);
                    loop_timer = HAL_GetTick() + 10000;
                    state = MAIN_GREEN;
                } else if (!normal && main) {
                    timer = HAL_GetTick() + 3000;
                    state = WAIT_CHECK;
                }
                break;

            case WAIT_CHECK:
                if (read_normal()) {
                    set_normal(0, 0, 1);
                    state = NORMAL_YELLOW;
                } else if (HAL_GetTick() >= timer) {
                    set_normal(0, 1, 0);
                    HAL_Delay(1000);

                    set_normal(1, 0, 0);
                    HAL_Delay(1000);

                    set_main(0, 0, 1);
                    state = IDLE;
                }
                break;
        }

        prev_normal = normal;
        prev_main = main;
        HAL_Delay(10);
    }
}

void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Configure the main internal regulator output voltage
  */
  __HAL_RCC_PWR_CLK_ENABLE();
  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_HSI;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0) != HAL_OK)
  {
    Error_Handler();
  }
}

void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
  /* USER CODE END Error_Handler_Debug */
}

 ```



youtube link presentation:[YOUTUBE](https://youtu.be/iW0UrDKUFU4)
