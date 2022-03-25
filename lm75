#include "stm32f10x.h"                  // Device header

void UART_Transmit(char *string)										//Function for transmission of the data
{
	while(*string)
	{
		while(!(USART1->SR & 0x00000040));
		USART_SendData(USART1,*string);
		*string++;
	}
}

	float adcValue=0;
	float Tmax;
	static char recievedData='1';
	char sentData[20];
	static uint8_t dataBuffer[2];
	static float Tsensor;
  int i;

int main(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;								  	//Structures
	ADC_InitTypeDef ADC_InitStructure;
	USART_InitTypeDef USART_InitStructure;
	TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
	I2C_InitTypeDef I2C_InitStructure;
	
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1 | RCC_APB1Periph_TIM2, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOB | RCC_APB2Periph_ADC1 | RCC_APB2Periph_AFIO | RCC_APB2Periph_USART1, ENABLE);
	
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;								//Configure pin A0 as Analog Input
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;  			 				//Configure pin A1 as output
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1;              
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2;												//Configure pin A2 (Input)
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;					                //Configure pins B6 and B7 as SCL and SDA respectively.
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_9;								//Configue UART TX - UART module's RX should be connected to this pin
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;							//pin A9
	GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_AF_PP;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_10;								//Configue UART RX - UART module's TX should be connected to this pin
	GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_IN_FLOATING;					                //pin A10
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	TIM_TimeBaseStructure.TIM_Period = 35999;								//Configure the timer
	TIM_TimeBaseStructure.TIM_Prescaler = 9;
	TIM_TimeBaseStructure.TIM_ClockDivision = 0;
	TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);
	TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
	TIM_Cmd(TIM2, ENABLE);
	
	// Configure ADC mode (Independent | RegSimult | FastInterl | etc.)
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;						
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	// Enable/disable external conversion trigger (EXTI | TIM | etc.)
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	// Configure data alignment (Right | Left)
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	// Set the number of channels to be used and initialize ADC
	ADC_InitStructure.ADC_NbrOfChannel = 1;
	ADC_Init(ADC1, &ADC_InitStructure);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_7Cycles5);
	ADC_Cmd(ADC1, ENABLE);
	
	ADC_ResetCalibration(ADC1);
	while(ADC_GetResetCalibrationStatus(ADC1));
	ADC_StartCalibration(ADC1);
	while(ADC_GetCalibrationStatus(ADC1));
	// Start the conversion
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
	
	USART_InitStructure.USART_BaudRate = 9600;								//Configure the USART parameters
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;
	USART_InitStructure.USART_StopBits = USART_StopBits_1;
	USART_InitStructure.USART_Parity = USART_Parity_No;
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_Init(USART1, &USART_InitStructure);
	
	USART_Cmd(USART1, ENABLE);
	
	I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
	I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;
	I2C_InitStructure.I2C_OwnAddress1 = 0x00;
	I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;
	I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
	I2C_InitStructure.I2C_ClockSpeed = 100000;
	I2C_Init(I2C1, &I2C_InitStructure);
	
	I2C_Cmd(I2C1, ENABLE);

	
	while(1)
	{	
		adcValue=ADC_GetConversionValue(ADC1);								//Read the ADC value
       if(GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_2)) 
		{												//Read the ADC value
		Tmax=(int)(adcValue*0.01220703125);
		}
		
		for(i =20000;i>0;i--)									                //Wait for an input from the terminal
		{
			recievedData=USART_ReceiveData(USART1);							//Recieve data from the USART1
		}
		
		// Wait if busy
		while (I2C_GetFlagStatus(I2C1, I2C_FLAG_BUSY));
		// Generate START condition
		I2C_GenerateSTART(I2C1, ENABLE);
		while (!I2C_GetFlagStatus(I2C1, I2C_FLAG_SB));
		// Send device address for read
		I2C_Send7bitAddress(I2C1, 10010001, I2C_Direction_Receiver);
		while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED));
		// Read the first data
		while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_RECEIVED));
		dataBuffer[0] = I2C_ReceiveData(I2C1);
		// Disable ACK and generate stop condition
		I2C_AcknowledgeConfig(I2C1, DISABLE);
		I2C_GenerateSTOP(I2C1, ENABLE);
		// Read the second data
		while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_RECEIVED));
		dataBuffer[1] = I2C_ReceiveData(I2C1);
		
		Tsensor= (dataBuffer[0] << 8) | (dataBuffer[1] & 0xff);				                //Combine the most significant bits and least signifiacnt bits
		Tsensor= Tsensor/256;										//Into a single variable and divide it by 256 (2^8)
		
		if(recievedData=='1' && Tmax<Tsensor)								//Check if the recievedData is one AND temeperature is above max
		{												//temperature.
			GPIO_SetBits(GPIOA, GPIO_Pin_1);							//If the conditions are satisfied turn the LED on
		}
		
		if(recievedData=='0' || Tmax>Tsensor)								//Check if the recievedData is zero OR temberature is below max
		{												//temperature
			GPIO_ResetBits(GPIOA, GPIO_Pin_1);							//If the conditions are satisfied turn the LED off
		}
	
		sprintf(sentData,"%f\r",Tsensor);								//Prepare data to be transmitted
		UART_Transmit(sentData);									//Transmit the data
		
		for(i=4000000;i>0;i--)									//A delay of ~1 second
		{
			__NOP();
		}
	}
}
