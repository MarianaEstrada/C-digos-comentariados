# Códigos comentariados

## LED ON 

Este progrma enciende el LED ubicado en el pin PA5

~~~

/**
 ******************************************************************************
 * @file           : main.c
 * @author         : Auto-generated by STM32CubeIDE
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * <h2><center>&copy; Copyright (c) 2019 STMicroelectronics.
 * All rights reserved.</center></h2>
 *
 * This software component is licensed by ST under BSD 3-Clause license,
 * the "License"; You may not use this file except in compliance with the
 * License. You may obtain a copy of the License at:
 *                        opensource.org/licenses/BSD-3-Clause
 *
 ******************************************************************************
 */

#include "stm32l476xx.h"

/*************************************************
* function declarations
*************************************************/
int main(void);

int main(void)
{

	// Se habilita el banco A 
	RCC->AHB2ENR = 0x00000001;

	// Se configura el pin PA5
	GPIOA->MODER &= 0xABFFFFFF;		// Se limpian los bits 10 y 11 correspondientes a P5 donde esta el LED
	GPIOA->MODER &= 0xFFFFF7FF;		// Se pone 01 en los bits 10 y 11 para que se configure como una salida 

	//Envia un alto lógico (1) al pin 5 del puerto A
	GPIOA->ODR |= 0x0020;	

	while(1)
	{
		
	}

	__asm__("NOP"); // Assembly inline can be used if needed
	return 0;
}

~~~~


## Button LED

Este programa  detecta cuando se oprimió el pulsador ubicado en el pin PC13 y apaga el LED ubicado en PA5 .

~~~
/**
 ******************************************************************************
 * @file           : main.c
 * @author         : Auto-generated by STM32CubeIDE
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * <h2><center>&copy; Copyright (c) 2019 STMicroelectronics.
 * All rights reserved.</center></h2>
 *
 * This software component is licensed by ST under BSD 3-Clause license,
 * the "License"; You may not use this file except in compliance with the
 * License. You may obtain a copy of the License at:
 *                        opensource.org/licenses/BSD-3-Clause
 *
 ******************************************************************************
 */

#include "stm32l476xx.h"

// Se define una variable LEDDELAY de 1seg
#define LEDDELAY    1000000

/*************************************************
* function declarations
*************************************************/
int main(void);

int main(void)
{
	// Se habilita el banco A y el banco C 
	RCC->AHB2ENR = 0x00000005;

	// Se configura el pin PA5
	GPIOA->MODER &= 0xABFFFFFF;		// Se limpian los bits 10 y 11 correspondientes a P5 donde esta el LED
	GPIOA->MODER &= 0xFFFFF7FF;		// Se pone 01 en los bits 10 y 11 para que se configure como una salida 

   // Se configura el pin PC13
	GPIOC->MODER &= 0xFFFFFFFF;		// Se limpian los bits 26 y 27 correspondientes a PC13  donde esta el pulsador
	GPIOC->MODER &= 0xF3FFFFFF;		// Se pone 00 en los bits 26 y 27 para que se configure como una ENTRADA

	while(1)
	{
  
  //Detecta si se oprimio el botón, si esto ocurre el LED se apaga.
		if(GPIOC->IDR == (uint16_t)(1 << 13)) {
    // Se enciende el LED
			GPIOA->ODR |= 0x0020;	
		} else {
    // Se apaga el LED
			GPIOA->ODR &= 0x0000;   
		}
	}

	__asm__("NOP"); // Assembly inline can be used if needed
	return 0;
}
~~~

## LED blink

Este programa hace que el LED ubicado en PA5 titile cada 1seg.

~~~
/**
 ******************************************************************************
 * @file           : main.c
 * @author         : Auto-generated by STM32CubeIDE
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * <h2><center>&copy; Copyright (c) 2019 STMicroelectronics.
 * All rights reserved.</center></h2>
 *
 * This software component is licensed by ST under BSD 3-Clause license,
 * the "License"; You may not use this file except in compliance with the
 * License. You may obtain a copy of the License at:
 *                        opensource.org/licenses/BSD-3-Clause
 *
 ******************************************************************************
 */

#include "stm32l476xx.h"

// Se define una variable LEDDELAY de 1seg
#define LEDDELAY    1000000

/*************************************************
* function declarations
*************************************************/
int main(void);
//Se activa la función delay la cual tendrá como entrada una variable volatile de 32 bits.
void delay(volatile uint32_t s);

int main(void)
{

	// Se habilita el banco A 
	RCC->AHB2ENR = 0x00000001;

	// Se configura el pin PA5
	GPIOA->MODER &= 0xABFFFFFF;		// Se limpian los bits 10 y 11 correspondientes a P5 donde esta el LED
	GPIOA->MODER &= 0xFFFFF7FF;		// Se pone 01 en los bits 10 y 11 para que se configure como una salida 

	//Envia un alto lógico (1) al pin 5 del puerto A
	GPIOA->ODR |= 0x0020;			

	while(1)
	{
	
	// Se llama la función delay y va a tener como entrada la variable LEDDELAY que corresponde a 1seg.
		delay(LEDDELAY);
	// La función XOR permite que varie de 0 a 1 
		GPIOA->ODR ^= (1 << 5);  // Toggle LED
	}

	__asm__("NOP"); // Assembly inline can be used if needed
	return 0;
}

// Se define la función delay, la cual es una función de retraso.
void delay(volatile uint32_t s)
{
    for(s; s>0; s--){
        // Do nothing
    }
}
~~~

## Interrupción externa 

Este progama hace que el LED ubicado en el pin PA5 se encienda o se apague, cada vez que se oprime el botón ubicado en PC13 utilizando interrupciones externas.

~~~
 *
 * This software component is licensed by ST under BSD 3-Clause license,
 * the "License"; You may not use this file except in compliance with the
 * License. You may obtain a copy of the License at:
 *                        opensource.org/licenses/BSD-3-Clause
 *
 ******************************************************************************
 */

#include "stm32l476xx.h"

// Se define una variable LEDDELAY con un valor de 1000
#define LEDDELAY    1000

/*************************************************
* function declarations
*************************************************/
int main(void);

/*************************************************
* external interrupt handler
*************************************************/

// Se crea una función que se denomina hilo
void EXTI15_10_IRQHandler(void)
{
	// Verifica si la interrupción fue activada por PC13
	if(EXTI->PR1 & (1 <<13)) {    // PR1 es un registro de bandera 
		GPIOA->ODR ^= (uint16_t)(1 << 5); //Enciende el LED ubicado en pA5

		// Limpia la interrupción, la interrupción no limpia con ceros sino con uno.
		EXTI->PR1 = 0x00002000;
	}
}

int main(void)
{
	// Se habilita el banco A y C 
	RCC->AHB2ENR = 0x00000005;

	// Se configura el pin PA5
	GPIOA->MODER &= 0xABFFFFFF;		// Se limpian los bits 10 y 11 correspondientes a P5 donde esta el LED
	GPIOA->MODER &= 0xFFFFF7FF;		// Se pone 01 en los bits 10 y 11 para que se configure como una salida 

 	  // Se configura el pin PC13
	GPIOC->MODER &= 0xFFFFFFFF;		// Se limpian los bits 26 y 27 correspondientes a PC13  donde esta el pulsador
	GPIOC->MODER &= 0xF3FFFFFF;		// Se pone 00 en los bits 26 y 27 para que se configure como una ENTRADA
	
	// Para que la interrupción empiece a trabajar es necesario activar el SYSCFG 
	RCC->APB2ENR = 0x00000001; // Se habilita el bit 0 

	/* tie push button at PC13 to EXTI4 */
	// EXTI4 can be configured for each GPIO module.
	// EXTICR1: 0b XXXX XXXX XXXX 0000
	//             pin3 pin2 pin1 pin0
	//
	// Se va le va a decir al SYSCFG que se va trabajar una interrupción externa.
	//Se habilita la interrupción 3 porque es la que corresponde a PC13
	SYSCFG->EXTICR[3] |= 0x0020;	//Se pone 0010 de los bits de 4 a 7 
	//Se configura que la interrupción va a funcionar con flancos de subida
	EXTI->RTSR1 |= 0x00002000;	// Se pone un 1 en el bit 13
	// Se hace un enmascaramiento de la interrupción 
	EXTI->IMR1 |= 0x00002000;	
	// Se le da una prioridad a la interrupción nivel 1
	// EXTI15_10 es el nombre de la prioridad.
	NVIC->IP[EXTI15_10_IRQn] = 0x10; // El 00 y el 01 no se pueden usar porque pertenece al reset y donde inicia la memoria.
	// Se habilita la interrupción.
	NVIC_EnableIRQ(EXTI15_10_IRQn);



	while(1)
	{

	}

	__asm__("NOP"); // Assembly inline can be used if needed
	return 0;
}
~~~


## SPI 

~~~
/**
 ******************************************************************************
 * @file           : main.c
 * @author         : Auto-generated by STM32CubeIDE
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * <h2><center>&copy; Copyright (c) 2019 STMicroelectronics.
 * All rights reserved.</center></h2>
 *
 * This software component is licensed by ST under BSD 3-Clause license,
 * the "License"; You may not use this file except in compliance with the
 * License. You may obtain a copy of the License at:
 *                        opensource.org/licenses/BSD-3-Clause
 *
 ******************************************************************************
 */

//Se cargan las librerias que da el fabricante para el manejo de la pantalla de papel
#include <epaper.h>
#include "stm32l476xx.h"
#include "picture.h"



/*************************************************
* function declarations
*************************************************/
int main(void);
void spi_init(void);

int main(void)
{

	// Se define el SPI, para inicializarlo se debe saber que el reloj del micro es de 4MHz
	spi_init();

	//Full screen refresh
	EPD_HW_Init(); //Electronic paper initialization
	EPD_WhiteScreen_ALL(gImage_3); //Refresh the picture in full screen
	driver_delay_xms(3000);
	//EPD_WhiteScreen_ALL(gImage_2); //Refresh the picture in full screen
	//driver_delay_xms(3000);
	//EPD_WhiteScreen_ALL(gImage_3); //Refresh the picture in full screen
	//driver_delay_xms(3000);
	//EPD_WhiteScreen_ALL(gImage_4); //Refresh the picture in full screen
	//driver_delay_xms(3000);

	while(1)
	{

	}

	__asm__("NOP"); // Assembly inline can be used if needed
	return 0;
}

void spi_init(void)
{

	// Se activa el banco A, B y C
	RCC->AHB2ENR = 0x00000007;

	// Se activa el chip select (SPI1) por el puerto PB6
	GPIOB->MODER &= 0xFFFFDFFF;		// Se escribe 01 para configurarlo como una salida
	GPIOB->ODR |= (1 << 6); // Se escribe un alto logico (1) en el bit 6 del banco B.

	// Si se va a usar un pin para dato o control se usa el banco C 
		GPIOC->MODER &= 0xFFFF7FFF;		// Se pone 01 para configurarlo como una salida
	// SPI1 data pins setup
	// Se escribe 10 en los bits 5,6 y 7 para configurarlos como selector alterno (spi)
	// Se escribe 01 en el bit 9 para configurarlos como salida(reset)
	// Se escribe 00 en el bit 8 para configurarlos como entrada(busy)
	GPIOA->MODER &= 0xFFF4ABFF;
	GPIOA->PUPDR |= 0x00010000; //Se configura el pull up

	// Se activa el reloj del SPI en el bit 12
	RCC->APB2ENR |= (1 << 12);

	//Dado que se va a trabar en muy altas frecuencias (por encima de 1MHz) es necesario activar el OSPEEDR
	GPIOA->OSPEEDR |= 0x0000FC00; // Set pin 5/6/7 to very high speed mode (0b11)

	// choose AF5 for SPI1 in Alternate Function registers
	GPIOA->AFR[0] |= (0x5 << 20); // para el pin 5 se activa el reloj
	GPIOA->AFR[0] |= (0x5 << 24); // para el pin 6 se activa el MISO, aunque la pantalla no envia datos es necesario configurarla.
	GPIOA->AFR[0] |= (0x5 << 28); // para el pin 7 se activa el MOSI

	// Disable SPI1 and set the rest (no OR'ing)
        // Se configura la señal de reloj
	// baud rate - BR[2:0] is bit 5:3
	// fPCLK/2 se sabe que el reloj es de 4MHZ/2 =2Mz
	//SPI1->CR1 = (0 << 3); // Por lo anterior no se activa este comando porque el ya lo tiene predifinido en 2MHz
	
	// Ahora se van a configurar los modos, que puede ser de 8-16bits, se va a trabajar con 8 bits que es el que trae definido
	//SPI1->CR1 |= (1 << 11); // Si se fuera a trabajar con 16 se debe activar esta función

	// motion sensor expects clk to be high and
	//  transmission happens on the falling edge
	// Se va a configurar la polaridad y la fase
	// Se pone un 0 en el bit 1 (polaridad)
	SPI1->CR1 |= (0 << 1); // clk goes 1 when idle
	// Se pone un 0 en el bit 0 (fase)
	SPI1->CR1 |= (0 << 0); // first clock transaction

	// Se difine la forma en la que se quiere transmitir si con el bit más significativo o con el bit menos significativo.
	//Generalmente se traba con el más significativo
	// 0 - MSB transmitted first
	//SPI1->CR1 |= (0 << 7); // Si se quiere trabajar con el menos significativo.

	// Ahora se difine la forma como se quieren trasmitir los datos modo motorola o modo texas instruments (TI)
	// 0 - Motorola mode, 1 - TI mode
	// Es mejor trabajar con el modo motorola, que es el modo predefinido
	//SPI1->CR2 |= (1 << 4); // Si se quiere trabajar con TI
	
	// Cuando se trabaja con el modo motorola se define como quiere que trabaje el chip selec
	// software slave management - SSM bit 9
	SPI1->CR1 |= (1 << 9); // 1- ssm enabled
	// internal slave select - SSI bit 8
	SPI1->CR1 |= (1 << 8); // set ssi to 1

	// Se activa el modo en el que se quiere trabajar, en este caso es modo maestro
	// master config - MSTR bit 2
	SPI1->CR1 |= (1 << 2); // 1 - master mode
	
	// Se habilita el SPI
	// enable SPI - SPE bit 6
	SPI1->CR1 |= (1 << 6);
}

~~~

## I2C




