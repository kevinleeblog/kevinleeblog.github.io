---
layout: post
title: "3歲智商的掃地機器人"
auther: Kevin Lee
category: 
tags: [MCU_Design, Job_Logging]
subtitle:
visualworkflow: true
---

#### 為何做？

公司代理一個新的Image IC，之前的工程師是把這個IC應用在跑步機上用來偵測跑者速度，後來PM就想了一個應用就是Robot，延者牆壁走，要能避開障礙物以及模擬找到充電站的功能，整個規格就是這樣，接下來就是我自由發揮怎麼達到功能

#### 如何實現？

##### 機構和硬體

Robot的主體要能夠移動，但時間有限的情況下，去買了迷宮鼠當作Robot的本體
![3pi-usb-]({{site.baseurl}}/img/3pi-usb-.jpg)

這個迷宮鼠是外面用來比賽用
![images]({{site.baseurl}}/img/images.jpeg)

但我不是要去做迷宮鼠，我是要做掃地機器人的功能，所以除了迷宮鼠外，還另外外掛一個MCU Board，上面有Image IC以及把Active IR也加進來

實際成品大概就如下
![image-20191030114131906]({{site.baseurl}}/img/image-20191030114131906.png)

##### 韌體

迷宮鼠上的韌體主要就是兩個功能

1. 和MCU Board上通訊
2. 控制Robot的方向

MCU Board上的功能

1. 初始化Image IC和Active IR IC
2. 避障與辨識功能
3. 和迷宮鼠通訊

大致上的方塊圖如下
![image-20191030114818742]({{site.baseurl}}/img/image-20191030114818742.png)

要達到MCU Board上的所有複雜功能又要不能延遲，我是採用事件驅動的軟體架構

```C
typedef struct
{
	RobotFlowSequenceState_DefType state;
	RobotFlowSequenceEvent_DefType event;
	void (*func)(void);
}RobotFlowSequence_DefType;

RobotFlowSequence_DefType robotStateTrans[] =
{	
		{STATE_ROBOT_POWER_ON, NOT_EVENT, initMySetting},

		{STATE_INIT_MY_SETTING,EVENT_UART_RECECOMM_SEARCH_BEACON_1, receUartCommSearchBeacon1},
		{STATE_INIT_MY_SETTING,EVENT_UART_RECECOMM_SEARCH_BEACON_2, receUartCommSearchBeacon2},
		{STATE_INIT_MY_SETTING,EVENT_UART_RECECOMM_GOBACK_DOCKING, receUartCommGoBackDocking},

		{STATE_RECEV_COMM_GOBACK_DOCKING, NOT_EVENT, SwitchToAIRMode},
		{STATE_RECEV_COMM_SEARCH_BEACON_1, NOT_EVENT, SwitchToAIRMode},
		{STATE_RECEV_COMM_SEARCH_BEACON_2, NOT_EVENT, SwitchToAIRMode},

		{STATE_ROBOT_LOCATE_IN_DOCKING,NOT_EVENT,idle},
		{STATE_ROBOT_LOCATE_IN_DOCKING,EVENT_FINDING_WALL,findTheWall},

		{STATE_FOUND_THE_WALL_IN_DOCKING_TIMEOUT,NOT_EVENT,robotStartActing},
	
		{STATE_ROBOT_IR_RANGE_MIDDLE, NOT_EVENT,letRobotMovingForward},
		{STATE_ROBOT_TOO_CLOSE_RIGHT, NOT_EVENT, letRobotMovingLeft},	
};

int main(void)
 {
	uint8_t i;
    // initialize the device
    SYSTEM_Initialize();
    i2c_iqs622_setup();
    delayMS(50);
    PAC7025_1_Initialize();
    /* Wait for 3pi pololu */
    delayMS(100);
    debugMessage("Robot power on!\r\n");
    robotSequenceTranArrayNumber = getRobotSequenceTranArrayNumber();
    while(1)
    {
    	robotCurrentEvent = getRobotNewEvent();
    	for(i = 0; i < robotSequenceTranArrayNumber; i++)
    	{
    		if (robotStateTrans[i].state == robotCurrentState
    							&& robotStateTrans[i].event == robotCurrentEvent)
    		{
    			robotStateTrans[i].func();
    			break;
    		}
    	}
    }
    return -1;
}

```

> 現在越來越多公司是使用RTOS來完成複雜的應用
> 但要把RTOS Stack放到MCU，需要的RAM與ROM有一定要求
> 使用這種方式，8-bit MCU就夠用了



