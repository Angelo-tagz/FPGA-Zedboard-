/************************************************************************ 
 * File name: interrupt_counter_btn.c
 * Description: This code creates a Zynq embedded system that controls 
 * four LED-counters by push button triggered external interrupts.
 ************************************************************************/
 
#include "xparameters.h"
#include "xgpio.h"
#include "xscugic.h"
#include "xil_exception.h"
#include "xil_printf.h"

#define INTC_DEVICE_ID 		   XPAR_PS7_SCUGIC_0_DEVICE_ID
#define BTNS_DEVICE_ID		   XPAR_AXI_GPIO_0_DEVICE_ID
#define LEDS_DEVICE_ID		   XPAR_AXI_GPIO_1_DEVICE_ID
#define INTC_GPIO_INTERRUPT_ID XPAR_FABRIC_AXI_GPIO_0_IP2INTC_IRPT_INTR
#define BTN_INT 			   XGPIO_IR_CH1_MASK

XGpio      LEDInst;
XGpio      BTNInst;
XScuGic    INTCInst;
static int led_data;
static int btn_value;

static void BTN_Intr_Handler(void *baseaddr_p);
static int  InterruptSystemSetup(XScuGic *XScuGicInstancePtr);
static int  IntcInitFunction(u16 DeviceId, XGpio *GpioInstancePtr);

void BTN_Intr_Handler(void *InstancePtr)
{
	XGpio_InterruptDisable(&BTNInst, BTN_INT);

	if ((XGpio_InterruptGetStatus(&BTNInst) & BTN_INT) != BTN_INT)
	{
		return;
	}

	btn_value = XGpio_DiscreteRead(&BTNInst, 1);

	if(btn_value != 1)
	{
		led_data = led_data + btn_value;
	}
	else
	{
		led_data = 0;
	}

    XGpio_DiscreteWrite(&LEDInst, 1, led_data);

    (void)XGpio_InterruptClear(&BTNInst, BTN_INT);

    XGpio_InterruptEnable(&BTNInst, BTN_INT);
}

int main (void)
{
	int status;

	status = XGpio_Initialize(&LEDInst, LEDS_DEVICE_ID);

	if(status != XST_SUCCESS)
	{
		return XST_FAILURE;
	}

	status = XGpio_Initialize(&BTNInst, BTNS_DEVICE_ID);

	if(status != XST_SUCCESS)
	{
		return XST_FAILURE;
	}

	XGpio_SetDataDirection(&LEDInst, 1, 0x00);

	XGpio_SetDataDirection(&BTNInst, 1, 0xFF);


	status = IntcInitFunction(INTC_DEVICE_ID, &BTNInst);

	if(status != XST_SUCCESS)
	{
		return XST_FAILURE;
	}

	while(1);

	return 0;
}

int InterruptSystemSetup(XScuGic *XScuGicInstancePtr)
{
	XGpio_InterruptEnable(&BTNInst, BTN_INT);

	XGpio_InterruptGlobalEnable(&BTNInst);

	Xil_ExceptionRegisterHandler(XIL_EXCEPTION_ID_INT,
			 	 	 	 	 	 (Xil_ExceptionHandler)XScuGic_InterruptHandler,
			 	 	 	 	 	 XScuGicInstancePtr);

	Xil_ExceptionEnable();

	return XST_SUCCESS;
}

int IntcInitFunction(u16 DeviceId, XGpio *GpioInstancePtr)
{
	XScuGic_Config *IntcConfig;
	int status;

	IntcConfig = XScuGic_LookupConfig(DeviceId);

	status = XScuGic_CfgInitialize(&INTCInst, IntcConfig, IntcConfig->CpuBaseAddress);

	if(status != XST_SUCCESS)
	{
		return XST_FAILURE;
	}

	status = InterruptSystemSetup(&INTCInst);

	if(status != XST_SUCCESS)
	{
		return XST_FAILURE;
	}
	
	status = XScuGic_Connect(&INTCInst,
					  	  	 INTC_GPIO_INTERRUPT_ID,
					  	  	 (Xil_ExceptionHandler)BTN_Intr_Handler,
					  	  	 (void *)GpioInstancePtr);

	if(status != XST_SUCCESS)
	{
		return XST_FAILURE;
	}

	XGpio_InterruptEnable(GpioInstancePtr, 1);

	XGpio_InterruptGlobalEnable(GpioInstancePtr);

	XScuGic_Enable(&INTCInst, INTC_GPIO_INTERRUPT_ID);
	
	return XST_SUCCESS;
}

