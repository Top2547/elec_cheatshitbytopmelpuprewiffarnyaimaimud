# Elec practical Exam Cheatsheet


### 
## button matrix

```
void transitionPin() { 
// Disable the output first by setting the pin low 
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);  
// Reconfigure the pin as an input 
GPIO_InitStruct.Pin = GPIO_PIN_5; 
GPIO_InitStruct.Mode = GPIO_MODE_INPUT; 
GPIO_InitStruct.Pull = GPIO_NOPULL; 
// Adjust the pull-up/pull-down configuration if needed HAL_GPIO_Init(GPIOA, &GPIO_InitStruct); }
all gpio basic command

void switchBackToOutput() { 
// De-initialize or disable input mode 
HAL_GPIO_DeInit(GPIOA, GPIO_PIN_5);  
// Reconfigure the pin as an open-drain output 
GPIO_InitStruct.Pin = GPIO_PIN_5; 
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_OD; 
// Set as open-drain output 
GPIO_InitStruct.Pull = GPIO_NOPULL; 
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW; 
// Adjust speed if needed 
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct); 
}
```
Struct pin port button matrix
```
/* USER CODE BEGIN PV */
struct _ButMtx_Struct
{
GPIO_TypeDef* Port;
uint16_t Pin;
};

struct _ButMtx_Struct BMX_L[4] = {
{GPIOA,GPIO_PIN_9},
{GPIOC,GPIO_PIN_7},
{GPIOB,GPIO_PIN_6},
{GPIOA,GPIO_PIN_7}
};

struct _ButMtx_Struct BMX_R[4] = {
{GPIOB,GPIO_PIN_5},
{GPIOB,GPIO_PIN_4},
{GPIOB,GPIO_PIN_10},
{GPIOA,GPIO_PIN_8}
};

uint16_t ButtonState = 0;
/* USER CODE END PV */
```
For loop check button matrix
```
/* USER CODE BEGIN 4 */
void ButtonMatrixRead(){
static uint8_t X=0;
for(int i=0; i<4; i++)
{
if(HAL_GPIO_ReadPin(BMX_L[i].Port, BMX_L[i].Pin) == GPIO_PIN_RESET)
{ //ปุ่มถูกกด
ButtonState |= 1 << (i + (X * 4));
}
else
{
ButtonState &= ~(1 << (i + (X * 4)));
}
}
//set currentL to Hi-z (open drain)
HAL_GPIO_WritePin(BMX_R[X].Port, BMX_R[X].Pin, GPIO_PIN_SET);
//set nextL to low
uint8_t nextX = (X + 1) % 4;
HAL_GPIO_WritePin(BMX_R[nextX].Port, BMX_R[nextX].Pin, GPIO_PIN_RESET);
X = nextX;
}
/* USER CODE END 4 */
```
```
/* USER CODE BEGIN PFP */
void ButtonMatrixRead();
/* USER CODE END PFP */
```
Time stamp
```
/* USER CODE BEGIN WHILE */
while (1)
{
/* USER CODE END WHILE */

/* USER CODE BEGIN 3 */
static uint32_t BTMX_TimeStamp = 0;
if(HAL_GetTick() > BTMX_TimeStamp)
{
BTMX_TimeStamp = HAL_GetTick() + 25; //next scan in 25 ms
ButtonMatrixRead();
}
}
/* USER CODE END 3 */
```
## hall_gpio
LED ON
```
/* USER CODE BEGIN 2 */ 
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5,GPIO_PIN_SET);
/* USER CODE END 2 */
```
LED OFF
```
/* USER CODE BEGIN 2 */
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5,GPIO_PIN_RESET);
/* USER CODE END 2 */
```
LED ON500ms.OFF500ms.
```
/* USER CODE BEGIN 3 */
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
HAL_Delay(500); //On500msOff500ms
}
/* USER CODE END 3 */
```
Click on No click off
```
/* USER CODE BEGIN 3 */
GPIO_PinState B1 = HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13);
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, B1);
}
/* USER CODE END 3 */
```
Press S1 and S2
```
/* USER CODE BEGIN 3 */
GPIO_PinState S1 = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_8);
GPIO_PinState S2 = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_9);
GPIO_PinState D1 = S1 && S2;
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, D1);
```

**interrupt(slide3P.40)**
Set pin GPIO_exti
```
/* USER CODE BEGIN 4 */
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
if(GPIO_Pin == GPIO_PIN_13)
{
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5); //คำสั่งปรับเองชิ้วๆ
}
}
```

dac/adc**
ADC
```
/* USER CODE BEGIN PV */
struct _ADC_tag
{
ADC_ChannelConfTypeDef Config;
uint16_t data;
};
struct _ADC_tag ADC1_Channel[2] =
{
{
.Config.Channel = ADC_CHANNEL_1,
.Config.Rank = ADC_REGULAR_RANK_1,
.Config.SamplingTime = ADC_SAMPLETIME_640CYCLES_5,
.Config.SingleDiff = ADC_SINGLE_ENDED,
.Config.OffsetNumber = ADC_OFFSET_NONE,
.Config.Offset = 0,
.data = 0
},
{
.Config.Channel = ADC_CHANNEL_TEMPSENSOR_ADC1,
.Config.Rank = ADC_REGULAR_RANK_1,
.Config.SamplingTime = ADC_SAMPLETIME_640CYCLES_5,
.Config.SingleDiff = ADC_SINGLE_ENDED,
.Config.OffsetNumber = ADC_OFFSET_NONE,
.Config.Offset = 0,
.data = 0
}
};
```
```
/* USER CODE BEGIN PFP */
void ADC_Read_blocking();
/* USER CODE END PFP */
/* USER CODE BEGIN 4 */
void ADC_Read_blocking()
{
static uint32_t TimeStamp = 0;
if( HAL_GetTick()<TimeStamp) return;
TimeStamp = HAL_GetTick()+500;
for(int i=0;i<2;i++)
{
HAL_ADC_ConfigChannel(&hadc1, &ADC1_Channel[i].Config);
HAL_ADC_Start(&hadc1);
HAL_ADC_PollForConversion(&hadc1, 100);
ADC1_Channel[i].data = HAL_ADC_GetValue(&hadc1);
HAL_ADC_Stop(&hadc1);
}
}
/* USER CODE END 4 */
```
```
/* USER CODE BEGIN 2 */
HAL_ADCEx_Calibration_Start(&hadc1, ADC_SINGLE_ENDED);
/* USER CODE END 2 */
/* USER CODE BEGIN 3 */
ADC_Read_blocking();
}
/* USER CODE END 3 */
```
DAC
```
/* USER CODE BEGIN PV */
uint16_t DAC_Output=0;
/* USER CODE END PV */

/* USER CODE BEGIN PFP */
void DAC_Update();
/* USER CODE END PFP */

/* USER CODE BEGIN 2 */
HAL_DAC_SetValue(&hdac1, DAC_CHANNEL_1, DAC_ALIGN_12B_R, 2048);
HAL_DAC_Start(&hdac1, DAC_CHANNEL_1);
/* USER CODE END 2 */
```
```
/* USER CODE BEGIN 4 */
void DAC_Update()
{
static uint32_t timeStamp =0;
if(HAL_GetTick()>timeStamp)
{
timeStamp = HAL_GetTick()+500;
HAL_DAC_SetValue(&hdac1, DAC_CHANNEL_1, DAC_ALIGN_12B_R, DAC_Output);
}
}

/* USER CODE BEGIN 3 */
DAC_Update();
}
/* USER CODE END 3 */
```
