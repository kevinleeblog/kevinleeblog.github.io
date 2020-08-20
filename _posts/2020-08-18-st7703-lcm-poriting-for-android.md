---
layout: post
title: "LCM Module ST7703 porting to android"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle:
visualworkflow: true
---

這文件是描述我工作過程中使用Qualcomm msm8953點亮ST7703 LCM的過程，可能是新手的運氣也可能是我帥氣吧，第一次就順利完成點亮工作。

剛拿到新面板時，首先要先和供應商FAE確認這快面板是否需要init code去初始化以及面板的Hsync, Vsync, HBP...等參數。
面板的參數是要給80-NH713-1_H_DSI_Timing_Parameters1 excel計算

所以首先就先計算參數吧，下圖是和廠商要的面板資料
點亮螢幕有分兩塊lk和kernel，都需個別加入點亮螢幕的程式進去

## 1) 80-NH713-1_H_DSI_Timing_Parameters1計算

![image-20200818100401236]({{site.baseurl}}/img/image-20200818100401236.png)

![image-20200818095508146]({{site.baseurl}}/img/image-20200818095508146.png)  



把上述資料填到80-NH713-1_H_DSI_Timing_Parameters1

![image-20200818100029042]({{site.baseurl}}/img/image-20200818100029042.png)

到**DSI PHY timing setting**標籤頁，按下Crtl+J與Ctrl+K來計算

![image-20200818105132327]({{site.baseurl}}/img/image-20200818105132327.png)

由上圖DSI PHY register可得

<PanelTimings>"0x87, 0x1c, 0x12, 0x00, 0x42, 0x44, 0x18, 0x20, 0x17, 0x03, 0x04, 0x00"</PanelTimings>

由上圖T_CLK_POST與T_CLK_PRE可得

 <TClkPost>0x04</TClkPost>
<TClkPre>0x1b</TClkPre>

![image-20200818105402224]({{site.baseurl}}/img/image-20200818105402224.png)

由上圖DSI PHY 2.x.x registers轉換成程式為

```c
static const uint32_t st7703_720p_video_timings[] = {
       0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
        0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
       0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
       0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
       0x1e ,0x0e ,0x04 ,0x05 ,0x02 ,0x03 ,0x04 ,0xa0
};

```

> DSI PHY 2.0.0參數只需要加在lk就好，Kernel不需要此參數

## 2) LCM Init code

這塊LCM使用的controller為ST7703需要放入init code，所以需要和廠商拿調適為本面板使用的init code程式，然後自行轉換對應的格式
下頁為**LCM Power on command**

```c

//======================Initial Code



DSI_CMD(0x04,0xB9); /// Set EXTC

DSI_PA(0xF1);   //1

DSI_PA(0x12);   //2

DSI_PA(0x83);   //3



DSI_CMD(0x1C,0xBA); // Set DSI	

DSI_PA(0x33);	//1 //3Lane:0x32 4Lane:0x33

DSI_PA(0x81);	//2

DSI_PA(0x05);	//3

DSI_PA(0xF9);	//4

DSI_PA(0x0E);	//5

DSI_PA(0x0e);	//6

DSI_PA(0x20);	//7

DSI_PA(0x00);	//8

DSI_PA(0x00);	//9

DSI_PA(0x00);	//10

DSI_PA(0x00);	//11

DSI_PA(0x00);	//12

DSI_PA(0x00);	//13

DSI_PA(0x00);	//14

DSI_PA(0x44);	//15

DSI_PA(0x25);	//16

DSI_PA(0x00);	//17

DSI_PA(0x91);	//18

DSI_PA(0x0a);	//19

DSI_PA(0x00);	//20

DSI_PA(0x00);	//21

DSI_PA(0x02);	//22

DSI_PA(0x4F);	//23

DSI_PA(0x11);	//24

DSI_PA(0x00);	//25

DSI_PA(0x00);	//26

DSI_PA(0x37);	//27



DSI_CMD(0x05,0xB8); //ECP

DSI_PA(0x28);//POWER_IC 25  3POWER 75

DSI_PA(0x22);

DSI_PA(0x20);

DSI_PA(0x03);



DSI_CMD(0x0B,0xB3); //SET RGB	

DSI_PA(0x10); //1	

DSI_PA(0x10); //2	

DSI_PA(0x05); //3	

DSI_PA(0x05); //4	

DSI_PA(0x03); //5	

DSI_PA(0xFF); //6

DSI_PA(0x00); //7

DSI_PA(0x00); //8

DSI_PA(0x00); //9

DSI_PA(0x00); //10



DSI_CMD(0x0A,0xC0); //SCR

DSI_PA(0x70); //1

DSI_PA(0x73); //2

DSI_PA(0x50); //3

DSI_PA(0x50); //4

DSI_PA(0x00); //5

DSI_PA(0x00); //6

DSI_PA(0x08); //7

DSI_PA(0x70); //8

DSI_PA(0x00); //9



DSI_CMD(0x02,0xBC); ///Set VDC

DSI_PA(0x4E);   //1 



DSI_CMD(0x02,0xCC); ///Set Panel

DSI_PA(0x0B);   //1 //forward:0x0B backward:0x07



DSI_CMD(0x02,0xB4); ///Set Panel inversion

DSI_PA(0x80);   //1



DSI_CMD(0x04,0xB2); /// Set RSO

DSI_PA(0xFA);   //1 1480=480+250*4

DSI_PA(0x12);   //2 720RGB

DSI_PA(0xF0);   //3



DSI_CMD(0x0F,0xE3); /// Set EQ

DSI_PA(0x07);   //1  PNOEQ

DSI_PA(0x07);   //2  NNOEQ

DSI_PA(0x0B);   //3  PEQGND

DSI_PA(0x0B);   //4  NEQGND

DSI_PA(0x03);   //5  PEQVCI

DSI_PA(0x0B);   //6  NEQVCI

DSI_PA(0x00);   //7  PEQVCI1

DSI_PA(0x00);   //8  NEQVCI1

DSI_PA(0x00);   //9  VCOM_PULLGND_OFF

DSI_PA(0x00);   //10 VCOM_PULLGND_OFF

DSI_PA(0xFF);   //11 VCOM_IDLE_ON

DSI_PA(0x00);   //12 

DSI_PA(0xC0);   //13 defaut C0 ESD detect function

DSI_PA(0x10);   //14 SLPOTP 



DSI_CMD(0x0D,0xC1); ///Set POWER

DSI_PA(0x54);   //1 VBTHS VBTLS

DSI_PA(0x00);	//2

DSI_PA(0x1E);   //3 VSPR

DSI_PA(0x1E);   //4 VSNR

DSI_PA(0x77);   //5 VSP VSN	

DSI_PA(0xF1);   //6 APS	

DSI_PA(0xCC);   //7 VGH VGL

DSI_PA(0xDD);   //8 VGH VGL

DSI_PA(0x67);   //9 VGH2 VGL2

DSI_PA(0x77);   //10 VGH2 VGL2

DSI_PA(0x33);   //11 VGH3 VGL3

DSI_PA(0x33);   //12 VGH3 VGL3



DSI_CMD(0x03,0xB5); ///Set POWER

DSI_PA(0x0D);   //1 vref

DSI_PA(0x0D);   //2 nvref



DSI_CMD(0x03,0xB6); ///Set VCOM

DSI_PA(0x37);   //1 F_VCOM 	  30

DSI_PA(0x37);   //2 B_VCOM



DSI_CMD(0x04,0xBF); ///Set PCR

DSI_PA(0x02);  //1

DSI_PA(0x11);  //2

DSI_PA(0x00);  //3



DSI_CMD(0x40,0xE9); //GIP

DSI_PA(0xC2);  //1  PANSEL			 C2

DSI_PA(0x10);  //2  SHR_0[11:8]		  10

DSI_PA(0x07);  //3  SHR_0[7:0]		  07

DSI_PA(0x05);  //4  SHR_1[11:8]		  05

DSI_PA(0xC5);  //5  SHR_1[7:0]  	  D7

DSI_PA(0x87);  //6  SPON[7:0]

DSI_PA(0x90);  //7  SPOFF[7:0]

DSI_PA(0x12);  //8  SHR0_1[3:0], SHR0_2[3:0]

DSI_PA(0x31);  //9  SHR0_3[3:0], SHR1_1[3:0]

DSI_PA(0x23);  //10  SHR1_2[3:0], SHR1_3[3:0]

DSI_PA(0x47);  //11  SHP[3:0], SCP[3:0]

DSI_PA(0x84);  //12  CHR[7:0]		84

DSI_PA(0x87);  //13  CON[7:0]

DSI_PA(0x90);  //14  COFF[7:0]

DSI_PA(0x37);  //15  CHP[3:0], CCP[3:0]

DSI_PA(0x38);  //16  USER_GIP_GATE[7:0]

DSI_PA(0x0C);  //17  CGTS_L[21:16]          //COS19, COS20

DSI_PA(0x00);  //18  CGTS_L[15:8]

DSI_PA(0x18);  //19  CGTS_L[7:0]            //COS4,COS5

DSI_PA(0x00);  //20  CGTS_INV_L[21:16]

DSI_PA(0x00);  //21  CGTS_INV_L[15:8]

DSI_PA(0x00);  //22  CGTS_INV_L[7:0]

DSI_PA(0x0C);  //23  CGTS_R[21:16]          //COS19, COS20	

DSI_PA(0x00);  //24  CGTS_R[15:8]

DSI_PA(0x18);  //25  CGTS_R[7:0]            //COS4,COS5	

DSI_PA(0x00);  //26  CGTS_INV_R[21:16]

DSI_PA(0x00);  //27  CGTS_INV_R[15:8]

DSI_PA(0x00);  //28  CGTS_INV_R[7:0]

DSI_PA(0x88);  //29  COS1_L[3:0], COS2_L[3:0]     // VGL    VGL

DSI_PA(0x87);  //30  COS3_L[3:0], COS4_L[3:0]     // VGL  	STVR4

DSI_PA(0x57);  //31  COS5_L[3:0], COS6_L[3:0],    // STVR3 	CLK4

DSI_PA(0x53);  //32  COS7_L[3:0], COS8_L[3:0],    // CLK3	CLK2

DSI_PA(0x18);  //33  COS9_L[3:0], COS10_L[3:0]    // CLK1	X

DSI_PA(0x88);  //34  COS11_L[3:0], COS12_L[3:0]   // VGL	X

DSI_PA(0x88);  //35  COS13_L[3:0], COS14_L[3:0],  // X      X

DSI_PA(0x88);  //36  COS15_L[3:0], COS16_L[3:0],  // X      X

DSI_PA(0x88);  //37  COS17_L[3:0], COS18_L[3:0],  // X      X

DSI_PA(0x13);  //38  COS19_L[3:0], COS20_L[3:0],  // STVR1	STVR2	

DSI_PA(0x88);  //39  COS21_L[3:0], COS22_L[3:0],		

DSI_PA(0x88);  //40  COS1_R[3:0], COS2_R[3:0]     // VGL	VGL

DSI_PA(0x86);  //41  COS3_R[3:0], COS4_R[3:0]     // VGL	STVL4

DSI_PA(0x46);  //42  COS5_R[3:0], COS6_R[3:0]     // STVL3	CLK4

DSI_PA(0x42);  //43  COS7_R[3:0], COS8_R[3:0]     // CLK3	CLK2

DSI_PA(0x08);  //44  COS9_R[3:0], COS10_R[3:0]    // CLK1	X

DSI_PA(0x88);  //45  COS11_R[3:0], COS12_R[3:0]   // VGL    X

DSI_PA(0x88);  //46  COS13_R[3:0], COS14_R[3:0]   // X      X

DSI_PA(0x88);  //47  COS15_R[3:0], COS16_R[3:0]   // X      X

DSI_PA(0x88);  //48  COS17_R[3:0], COS18_R[3:0]   // X      X

DSI_PA(0x02);  //49  COS19_R[3:0], COS20_R[3:0]   // STVL1	STVL2	

DSI_PA(0x88);  //50  COS21_R[3:0], COS22_R[3:0],

DSI_PA(0x00);  //51  TCONOPTION

DSI_PA(0x00);  //52  OPTION

DSI_PA(0x00);  //53  OTPION

DSI_PA(0x00);  //54  OPTION

DSI_PA(0x00);  //55  CHR2

DSI_PA(0x00);  //56  CON2

DSI_PA(0x00);  //57  COFF2

DSI_PA(0x00);  //58  CHP2,CCP2

DSI_PA(0x00);  //59  CKS 20 19 18 17

DSI_PA(0x00);  //60  CKS 16 15 14 13

DSI_PA(0x00);  //61  CKS 8~1

DSI_PA(0x00);  //62  COFF[9:8]   CON[9:8]    SPOFF[9:8]    SPON[9:8]

DSI_PA(0x00);  //63  COFF2[9:8]  CON2[9:8]



DSI_CMD(0x3E,0xEA); //GIP

DSI_PA(0x02); //1  ys2_sel[1:0]

DSI_PA(0x21); //2  user_gip_gate1[7:0]

DSI_PA(0x00); //3  ck_all_on_width1[5:0]

DSI_PA(0x00); //4  ck_all_on_width2[5:0]

DSI_PA(0x00); //5  ck_all_on_width3[5:0]

DSI_PA(0x00); //6  ys_flag_period[7:0]

DSI_PA(0x00); //7  ys_2

DSI_PA(0x00); //8  user_gip_gate1_2[7:0]

DSI_PA(0x00); //9  ck_all_on_width1_2[5:0]

DSI_PA(0x00); //10 ck_all_on_width2_2[5:0]

DSI_PA(0x00); //11 ck_all_on_width3_2[5:0]

DSI_PA(0x00); //12 ys_flag_period_2[7:0]

DSI_PA(0x88); //13 COS1_L[3:0], COS2_L[3:0]     // VGL	    VGL	

DSI_PA(0x80); //14 COS3_L[3:0], COS4_L[3:0]     // VGL		STVR4

DSI_PA(0x24); //15 COS5_L[3:0], COS6_L[3:0],    // STVR3	CLK4

DSI_PA(0x60); //16 COS7_L[3:0], COS8_L[3:0],    // CLK3		CLK2

DSI_PA(0x28); //17 COS9_L[3:0], COS10_L[3:0]    // CLK1	    X

DSI_PA(0x88); //18 COS11_L[3:0], COS12_L[3:0]   // VGL		X

DSI_PA(0x88); //19 COS13_L[3:0], COS14_L[3:0],  // X        X

DSI_PA(0x88); //20 COS15_L[3:0], COS16_L[3:0],  // X		X

DSI_PA(0x88); //21 COS17_L[3:0], COS18_L[3:0],  // X		X

DSI_PA(0x64); //22 COS19_L[3:0], COS20_L[3:0],  // STVR1	STVR2	

DSI_PA(0x88); //23 COS21_L[3:0], COS22_L[3:0]		

DSI_PA(0x88); //24 COS1_R[3:0], COS2_R[3:0]     // VGL  	VGL	

DSI_PA(0x81); //25 COS3_R[3:0], COS4_R[3:0]     // VGL		STVL4

DSI_PA(0x35); //26 COS5_R[3:0], COS6_R[3:0]     // STVL3    CLK4

DSI_PA(0x71); //27 COS7_R[3:0], COS8_R[3:0]     // CLK3		CLK2

DSI_PA(0x38); //28 COS9_R[3:0], COS10_R[3:0]    // CLK1	    X

DSI_PA(0x88); //29 COS11_R[3:0], COS12_R[3:0]   // VGL      X

DSI_PA(0x88); //30 COS13_R[3:0], COS14_R[3:0]   // X        X

DSI_PA(0x88); //31 COS15_R[3:0], COS16_R[3:0]   // X        X

DSI_PA(0x88); //32 COS17_R[3:0], COS18_R[3:0]   // X        X

DSI_PA(0x75); //33 COS19_R[3:0], COS20_R[3:0]   // STVL1	STVL2

DSI_PA(0x88); //34 COS21_R[3:0], COS22_R[3:0]

DSI_PA(0x23); //35 EQOPT , EQ_SEL	

DSI_PA(0x10); //36 EQ_DELAY[7:0]	

DSI_PA(0x00); //37 EQ_DELAY_HSYNC [3:0]

DSI_PA(0x00); //38 HSYNC_TO_CL1_CNT9[8]

DSI_PA(0x08); //39 HSYNC_TO_CL1_CNT9[7:0]	

DSI_PA(0x00); //40 HIZ_L	

DSI_PA(0x00); //41 HIZ_R	

DSI_PA(0x00); //42 CKS_GS[21:16]	

DSI_PA(0x00); //43 CKS_GS[15:8]	

DSI_PA(0x00); //44 CKS_GS[7:0]	

DSI_PA(0x00); //45 CK_MSB_EN[21:16]	

DSI_PA(0x00); //46 CK_MSB_EN[15:8]	

DSI_PA(0x00); //47 CK_MSB_EN[7:0]	

DSI_PA(0x00); //48 CK_MSB_EN_GS[21:16]	

DSI_PA(0x00); //49 CK_MSB_EN_GS[15:8]

DSI_PA(0x00); //50 CK_MSB_EN_GS[7:0]

DSI_PA(0x00); //51 SHR2[12:8]

DSI_PA(0x00); //52 SHR2[7:0]

DSI_PA(0x00); //53 SHR2_1[3:0] / SHR2_2[3:0]

DSI_PA(0x00); //54 SHR2_3[3:0]

DSI_PA(0x40); //55 SHP1[3:0]/x

DSI_PA(0x87); //56 SPON1[7:0]

DSI_PA(0x90); //57 SPOFF1[7:0]

DSI_PA(0x00); //58 SHP2[3:0]

DSI_PA(0x00); //59 SPON2[7:0]

DSI_PA(0x00); //60 SPOFF2[7:0]

DSI_PA(0x00); //61 SPOFF2[9:8]/SPON2[9:8]/SPOFF1[9:8]/SPON1[9:8]



DSI_CMD(0x23,0xE0); ///Set Gamma徹傾OK

DSI_PA(0x00);  //1

DSI_PA(0x1A);  //2

DSI_PA(0x23);  //3

DSI_PA(0x27);  //4

DSI_PA(0x28);  //5

DSI_PA(0x3A);  //6

DSI_PA(0x50);  //7

DSI_PA(0x4E);  //8

DSI_PA(0x07);  //9

DSI_PA(0x0B);  //10

DSI_PA(0x0C);  //11

DSI_PA(0x0F);  //12

DSI_PA(0x11);  //13

DSI_PA(0x10);  //14

DSI_PA(0x13);  //15

DSI_PA(0x19);  //16

DSI_PA(0x1A);  //17

DSI_PA(0x00);  //1

DSI_PA(0x1A);  //2

DSI_PA(0x23);  //3

DSI_PA(0x27);  //4

DSI_PA(0x28);  //5

DSI_PA(0x3A);  //6

DSI_PA(0x50);  //7

DSI_PA(0x4E);  //8

DSI_PA(0x07);  //9

DSI_PA(0x0B);  //10

DSI_PA(0x0C);  //11

DSI_PA(0x0F);  //12

DSI_PA(0x11);  //13

DSI_PA(0x10);  //14

DSI_PA(0x13);  //15

DSI_PA(0x19);  //16

DSI_PA(0x1A);  //17



		

DSI_CMD(0x01,0x11); ////Sleep Out

DelayX1ms(250);

		

DSI_CMD(0x01,0x29); ///Display On

DelayX1ms(50);
```

**LCM Power off command**

```c
void PowerOffSequence() 
{ 
      DCS_Short_Write_NP(0x28); 
      Delay(200); 
      DCS_Short_Write_NP(0x10); 
      Delay(150); 
      Set_STANDBY();//Video transfer stop 
      Delay(50); 
      Set_RESET(1,0);//MIPI RESET 1, LCD RESET 0 
      Delay(50); 
      Set_RESET(0,0);//MIPI RESET 0, LCD RESET 0 
      Delay(50); 
      Set_POWER(1,1,0,1);//1.8V ON, 2.8V ON, 5V OFF, BL ON 
      Delay(150); 
      Set_BOOST(5.0, 5.0, 0x81, 50);//VDD, VEE, OFF:VDD->VEE, 10ms 
      Delay(50); 
      Set_POWER(1,0,0,1);//1.8V ON, 2.8V OFF, 5V OFF, BL ON 
      Delay(150); 
      Set_POWER(0,0,0,0);//1.8V OFF, 2.8V OFF, 5V OFF, BL OFF 
} 
```

把上述init code轉成qualcomm使用的格式

```
	<OnCommand>"0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x04, 0xB9, 0xF1, 0x12, 0x83, <!-- Set EXTC-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x1C, 0xBA, 0x33, 0x81, 0x05, 0xF9, 0x0E, 0x0E, 0x20, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x44, 0x25, 0x00, 0x91, 0x0a, 0x00, 0x00, 0x02, 0x4F, 0x11, 0x00, 0x00, 0x37 <!-- Set DSI-->                                        
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x05, 0xB8, 0x28, 0x22, 0x20, 0x03,    <!-- ECP--> 
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0B, 0xB3, 0x10, 0x10, 0x05, 0x05, 0x03, 0xFF, 0x00, 0x00, 0x00, 0x00, <!-- SET RGB-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0A, 0xC0, 0x70, 0x73, 0x50, 0x50, 0x00, 0x00, 0x08, 0x70, 0x00, <!-- SCR-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x02, 0xBC, 0x4E, <!-- SET VDC-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x02, 0xCC, 0x0B, <!-- Set Panel-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x02, 0xB4, 0x80, <!-- Set Panel inversion-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x04, 0xB2, 0xFA, 0x12, 0xF0, <!-- Set RSO-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0F, 0xE3, 0x07, 0x07, 0x0B, 0x0B, 0x03, 0x0B, 0x00, 0x00, 0x00, 0x00, 0xFF, 0x00, 0xC0, 0x10, <!-- Set EQ-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0D, 0xC1, 0x54, 0x00, 0x1E, 0x1E, 0x77, 0xF1, 0xCC, 0xDD, 0x67, 0x77, 0x33, 0x33, <!-- Set POWER-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x03, 0xB5, 0x0D, 0x0D, <!-- Set POWER-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x03, 0xB6, 0x37, 0x37, <!-- Set VCOM-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x04, 0xBF, 0x02, 0x11, 0x00, <!-- Set PCR-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x40, 0xE9, 0xC2, 0x10, 0x07, 0x05, 0xC5, 0x87 ,0x90, 0x12, 0x31 ,0x23 ,0x47 ,0x84 ,0x87 ,0x90 ,0x37 ,0x38 ,0x0C ,0x00 ,0x18 ,0x00 ,0x00 ,0x00 ,0x0C ,0x00 ,0x18 ,0x00 ,0x00 ,0x00, 0x88, 0x87, 0x57, 0x53, 0x18, 0x88, 0x88, 0x88, 0x88, 0x13, 0x88, 0x88, 0x86, 0x46, 0x42, 0x08, 0x88, 0x88, 0x88, 0x88, 0x02, 0x88, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, <!-- GIP-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x3E, 0xEA, 0x02 ,0x21 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00, 0x00 ,0x00 ,0x00 ,0x00 ,0x88 ,0x80 ,0x24 ,0x60 ,0x28 ,0x88 ,0x88 ,0x88 ,0x88 ,0x64, 0x88 ,0x88 ,0x81 ,0x35 ,0x71 ,0x38 ,0x88 ,0x88, 0x88 ,0x88 ,0x75, 0x88 ,0x23 ,0x10 ,0x00, 0x00 ,0x08 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x40 ,0x87 ,0x90 ,0x00, 0x00, 0x00, 0x00,<!-- GIP-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x23, 0xE0, 0x00, 0x1A, 0x23, 0x27, 0x28, 0x3A, 0x50, 0x4E, 0x07, 0x0B, 0x0C, 0x0F, 0x11, 0x10, 0x13, 0x19, 0x1A, 0x00, 0x1A, 0x23, 0x27, 0x28, 0x3A, 0x50, 0x4E, 0x07, 0x0B, 0x0C, 0x0F, 0x11, 0x10, 0x13, 0x19, 0x1A, <!-- Set Gamma-->
0x05, 0x01, 0x00, 0x00, 0xFA, 0x00, 0x02, 0x11, 0x00,  <!-- Sleep Out-->
0x05, 0x01, 0x00, 0x00, 0x32, 0x00, 0x02, 0x29, 0x00 <!-- Display On-->"</OnCommand>
```

```
<OffCommand>"05 01 00 00 C8 00 02 28 00,
	05 01 00 00 96 00 02 10 00"</OffCommand>
```

有了init code和面板參數後，就可以開始產生程式碼了

## 3) 製作LCM專用的xml檔

根據以上LCM參數與規格修改出xml
*device/qcom/common/display/tools/panel_st7703_720p_video.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
# Copyright (c) 2013, The Linux Foundation. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above
#       copyright notice, this list of conditions and the following
#       disclaimer in the documentation and/or other materials provided
#       with the distribution.
#     * Neither the name of The Linux Foundation nor the names of its
#       contributors may be used to endorse or promote products derived
#       from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED "AS IS" AND ANY EXPRESS OR IMPLIED
# WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT
# ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS
# BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
# BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
# WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
# OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
# IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
-->

<GCDB>
  <Version>"1.0"</Version>
  <PanelId>st7703-720p-video</PanelId>
  <PanelH>st7703_720p_video</PanelH>
  <PanelEntry>

	<!-- Panel configuration -->
	<PanelName>"st7703 720p video mode dsi panel"</PanelName>
	<PanelController>"mdss_dsi0"</PanelController>
	<PanelInterface>10</PanelInterface>
	<PanelType>0</PanelType>
	<PanelDestination>"DISPLAY_1"</PanelDestination>
	<PanelOrientation>0</PanelOrientation>
	<PanelFrameRate>60</PanelFrameRate>
	<PanelChannelId>0</PanelChannelId>
	<DSIVirtualChannelId>0</DSIVirtualChannelId>
	<PanelBroadcastMode>0</PanelBroadcastMode>
	<!-- Optional Panel configuration -->
	<!--BitClockFrequency>0</BitClockFrequency -->
	<DSIStream>0</DSIStream>
	<PanelCompatible>"qcom,mdss-dsi-panel"</PanelCompatible>
	<InterleaveMode>0</InterleaveMode>

	<!-- Panel Resolution -->
	<PanelWidth>720</PanelWidth>
	<PanelHeight>1480</PanelHeight>
	<HFrontPorch>40</HFrontPorch>
	<HBackPorch>40</HBackPorch>
	<HPulseWidth>8</HPulseWidth>
	<HSyncSkew>0</HSyncSkew>
	<VBackPorch>40</VBackPorch>
	<VFrontPorch>40</VFrontPorch>
	<VPulseWidth>8</VPulseWidth>
	<HLeftBorder>0</HLeftBorder>
	<HRightBorder>0</HRightBorder>
	<VTopBorder>0</VTopBorder>
	<VBottomBorder>0</VBottomBorder>
	<!-- Optional Panel resolution configuration -->
	<!--HActiveRes>0</HActiveRes>
	<VActiveRes>100</VActiveRes>
	<InvertDataPolarity>0</InvertDataPolarity>
	<InvertVsyncPolarity>0</InvertVsyncPolarity>
	<InvertHsyncPolarity>0</InvertHsyncPolarity -->

	<!-- Panel Color Information -->
	<ColorFormat>24</ColorFormat>
	<ColorOrder>0</ColorOrder>
	<UnderFlowColor>0xff</UnderFlowColor>
	<BorderColor>0</BorderColor>

	<!-- Panel Command information -->
	<OnCommand>"0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x04, 0xB9, 0xF1, 0x12, 0x83, <!-- Set EXTC-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x1C, 0xBA, 0x33, 0x81, 0x05, 0xF9, 0x0E, 0x0E, 0x20, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x44, 0x25, 0x00, 0x91, 0x0a, 0x00, 0x00, 0x02, 0x4F, 0x11, 0x00, 0x00, 0x37 <!-- Set DSI-->                                        
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x05, 0xB8, 0x28, 0x22, 0x20, 0x03,    <!-- ECP--> 
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0B, 0xB3, 0x10, 0x10, 0x05, 0x05, 0x03, 0xFF, 0x00, 0x00, 0x00, 0x00, <!-- SET RGB-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0A, 0xC0, 0x70, 0x73, 0x50, 0x50, 0x00, 0x00, 0x08, 0x70, 0x00, <!-- SCR-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x02, 0xBC, 0x4E, <!-- SET VDC-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x02, 0xCC, 0x0B, <!-- Set Panel-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x02, 0xB4, 0x80, <!-- Set Panel inversion-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x04, 0xB2, 0xFA, 0x12, 0xF0, <!-- Set RSO-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0F, 0xE3, 0x07, 0x07, 0x0B, 0x0B, 0x03, 0x0B, 0x00, 0x00, 0x00, 0x00, 0xFF, 0x00, 0xC0, 0x10, <!-- Set EQ-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x0D, 0xC1, 0x54, 0x00, 0x1E, 0x1E, 0x77, 0xF1, 0xCC, 0xDD, 0x67, 0x77, 0x33, 0x33, <!-- Set POWER-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x03, 0xB5, 0x0D, 0x0D, <!-- Set POWER-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x03, 0xB6, 0x37, 0x37, <!-- Set VCOM-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x04, 0xBF, 0x02, 0x11, 0x00, <!-- Set PCR-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x40, 0xE9, 0xC2, 0x10, 0x07, 0x05, 0xC5, 0x87 ,0x90, 0x12, 0x31 ,0x23 ,0x47 ,0x84 ,0x87 ,0x90 ,0x37 ,0x38 ,0x0C ,0x00 ,0x18 ,0x00 ,0x00 ,0x00 ,0x0C ,0x00 ,0x18 ,0x00 ,0x00 ,0x00, 0x88, 0x87, 0x57, 0x53, 0x18, 0x88, 0x88, 0x88, 0x88, 0x13, 0x88, 0x88, 0x86, 0x46, 0x42, 0x08, 0x88, 0x88, 0x88, 0x88, 0x02, 0x88, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, <!-- GIP-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x3E, 0xEA, 0x02 ,0x21 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00, 0x00 ,0x00 ,0x00 ,0x00 ,0x88 ,0x80 ,0x24 ,0x60 ,0x28 ,0x88 ,0x88 ,0x88 ,0x88 ,0x64, 0x88 ,0x88 ,0x81 ,0x35 ,0x71 ,0x38 ,0x88 ,0x88, 0x88 ,0x88 ,0x75, 0x88 ,0x23 ,0x10 ,0x00, 0x00 ,0x08 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x00 ,0x40 ,0x87 ,0x90 ,0x00, 0x00, 0x00, 0x00,<!-- GIP-->
0x39, 0x01, 0x00, 0x00, 0x00, 0x00, 0x23, 0xE0, 0x00, 0x1A, 0x23, 0x27, 0x28, 0x3A, 0x50, 0x4E, 0x07, 0x0B, 0x0C, 0x0F, 0x11, 0x10, 0x13, 0x19, 0x1A, 0x00, 0x1A, 0x23, 0x27, 0x28, 0x3A, 0x50, 0x4E, 0x07, 0x0B, 0x0C, 0x0F, 0x11, 0x10, 0x13, 0x19, 0x1A, <!-- Set Gamma-->
0x05, 0x01, 0x00, 0x00, 0xFA, 0x00, 0x02, 0x11, 0x00,  <!-- Sleep Out-->
0x05, 0x01, 0x00, 0x00, 0x32, 0x00, 0x02, 0x29, 0x00 <!-- Display On-->"</OnCommand>
	<OffCommand>"05 01 00 00 C8 00 02 28 00,
	05 01 00 00 96 00 02 10 00"</OffCommand>
	<OnCommandState>0</OnCommandState>
	<OffCommandState>1</OffCommandState>

	<!-- Video mode panel information -->
	<HSyncPulse>1</HSyncPulse>
	<HFPPowerMode>0</HFPPowerMode>
	<HBPPowerMode>0</HBPPowerMode>
	<HSAPowerMode>0</HSAPowerMode>
	<BLLPEOFPowerMode>1</BLLPEOFPowerMode>
	<BLLPPowerMode>1</BLLPPowerMode>
	<TrafficMode>2</TrafficMode>
	<DMADelayAfterVsync>0</DMADelayAfterVsync>
	<BLLPEOFPower>0x9</BLLPEOFPower>

	<!-- Lane Configuration -->
	<DSILanes>4</DSILanes>
	<DSILaneMap>0</DSILaneMap>
	<Lane0State>1</Lane0State>
	<Lane1State>1</Lane1State>
	<Lane2State>1</Lane2State>
	<Lane3State>1</Lane3State>

	<!-- Panel Timing -->
	<PanelTimings>"0x87, 0x1c, 0x12, 0x00, 0x42, 0x44, 0x18, 0x20, 0x17, 0x03, 0x04, 0x00"</PanelTimings>
	<DSIMDPTrigger>0</DSIMDPTrigger>
	<DSIDMATrigger>4</DSIDMATrigger>
	<TClkPost>0x04</TClkPost>
	<TClkPre>0x1b</TClkPre>

	<!-- Backlight -->
	<BLInterfaceType>1</BLInterfaceType>
	<BLMinLevel>1</BLMinLevel>
	<BLMaxLevel>4095</BLMaxLevel>
	<BLStep>100</BLStep>
	<BLPMICModel>"PMIC_8941"</BLPMICModel>
	<BLPMICControlType>0</BLPMICControlType>

	<!-- Panel Reset Sequence -->
	<ResetSequence>
		<PinState1>1</PinState1>
		<PulseWidth1>20</PulseWidth1>
		<PinState2>0</PinState2>
		<PulseWidth2>2</PulseWidth2>
		<PinState3>1</PinState3>
		<PulseWidth3>60</PulseWidth3>
		<EnableBit>2</EnableBit>
	</ResetSequence>

  </PanelEntry>
</GCDB>

```

產生dts和header

```
$ cd device/qcom/common/display/tools
$ perl parser.pl panel_st7703_720p_video.xml panel
```

當前目錄下會多兩個檔案
panel_st7703_720p_video.xml
panel_st7703_720p_video.h

## 4) lk修改

把panel_st7703_720p_video.h移到bootable/bootloader/lk/dev/gcdb/display/include/目錄下

根據Quectel_SC60_Display_Driver_Development_Guide_V1.1文件，產生出來的header還需要經過修改才能正確編譯，以下是經過修正後的版本
也把power off command也一併加進去

*bootable/bootloader/lk/dev/gcdb/display/include/panel_st7703_720p_video.h*

```c
/* Copyright (c) 2013, The Linux Foundation. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *  * Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in
 *    the documentation and/or other materials provided with the
 *    distribution.
 *  * Neither the name of The Linux Foundation nor the names of its
 *    contributors may be used to endorse or promote products derived
 *    from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
 * "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
 * LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
 * FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
 * COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
 * INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
 * BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS
 * OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
 * AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT
 * OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
 * SUCH DAMAGE.
 */

/*---------------------------------------------------------------------------
 * This file is autogenerated file using gcdb parser. Please do not edit it.
 * Update input XML file to add a new entry or update variable in this file
 * VERSION = "1.0"
 *---------------------------------------------------------------------------*/

#ifndef _PANEL_ST7703_720P_VIDEO_H_
#define _PANEL_ST7703_720P_VIDEO_H_
/*---------------------------------------------------------------------------*/
/* HEADER files                                                              */
/*---------------------------------------------------------------------------*/
#include "panel.h"

/*---------------------------------------------------------------------------*/
/* Panel configuration                                                       */
/*---------------------------------------------------------------------------*/
static struct panel_config st7703_720p_video_panel_data = {
	"qcom,mdss_dsi_st7703_720p_video", "dsi:0:", "qcom,mdss-dsi-panel",
	10, 0, "DISPLAY_1", 0, 0, 60, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, NULL
};

/*---------------------------------------------------------------------------*/
/* Panel resolution                                                          */
/*---------------------------------------------------------------------------*/
static struct panel_resolution st7703_720p_video_panel_res = {
	720, 1480, 40, 40, 8, 0, 40, 40, 8, 0, 0, 0, 0, 0, 0, 0, 0, 0
};

/*---------------------------------------------------------------------------*/
/* Panel color information                                                   */
/*---------------------------------------------------------------------------*/
static struct color_info st7703_720p_video_color = {
	24, 0, 0xff, 0, 0, 0
};

/*---------------------------------------------------------------------------*/
/* Panel on/off command information                                          */
/*---------------------------------------------------------------------------*/
static char st7703_720p_video_on_cmd0[] = {
	0x04, 0x00, 0x39, 0xC0,
	0xB9, 0xF1, 0x12, 0x83,

};

static char st7703_720p_video_on_cmd1[] = {
	0x1C, 0x00, 0x39, 0xC0,
	0xBA, 0x33, 0x81, 0x05,
	0xF9, 0x0E, 0x0E, 0x20,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x44,
	0x25, 0x00, 0x91, 0x0a,
	0x00, 0x00, 0x02, 0x4F,
	0x11, 0x00, 0x00, 0x37,
};

static char st7703_720p_video_on_cmd2[] = {
	0x05, 0x00, 0x39, 0xC0,
	0xB8, 0x28, 0x22, 0x20,
	0x03, 0xFF, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd3[] = {
	0x0B, 0x00, 0x39, 0xC0,
	0xB3, 0x10, 0x10, 0x05,
	0x05, 0x03, 0xFF, 0x00,
	0x00, 0x00, 0x00, 0xFF,
};

static char st7703_720p_video_on_cmd4[] = {
	0x0A, 0x00, 0x39, 0xC0,
	0xC0, 0x70, 0x73, 0x50,
	0x50, 0x00, 0x00, 0x08,
	0x70, 0x00, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd5[] = {
	0x02, 0x00, 0x39, 0xC0,
	0xBC, 0x4E, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd6[] = {
	0x02, 0x00, 0x39, 0xC0,
	0xCC, 0x0B, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd7[] = {
	0x02, 0x00, 0x39, 0xC0,
	0xB4, 0x80, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd8[] = {
	0x04, 0x00, 0x39, 0xC0,
	0xB2, 0xFA, 0x12, 0xF0,

};

static char st7703_720p_video_on_cmd9[] = {
	0x0F, 0x00, 0x39, 0xC0,
	0xE3, 0x07, 0x07, 0x0B,
	0x0B, 0x03, 0x0B, 0x00,
	0x00, 0x00, 0x00, 0xFF,
	0x00, 0xC0, 0x10, 0xFF,
};

static char st7703_720p_video_on_cmd10[] = {
	0x0D, 0x00, 0x39, 0xC0,
	0xC1, 0x54, 0x00, 0x1E,
	0x1E, 0x77, 0xF1, 0xCC,
	0xDD, 0x67, 0x77, 0x33,
	0x33, 0xFF, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd11[] = {
	0x03, 0x00, 0x39, 0xC0,
	0xB5, 0x0D, 0x0D, 0xFF,
};

static char st7703_720p_video_on_cmd12[] = {
	0x03, 0x00, 0x39, 0xC0,
	0xB6, 0x37, 0x37, 0xFF,
};

static char st7703_720p_video_on_cmd13[] = {
	0x04, 0x00, 0x39, 0xC0,
	0xBF, 0x02, 0x11, 0x00,

};

static char st7703_720p_video_on_cmd14[] = {
	0x40, 0x00, 0x39, 0xC0,
	0xE9, 0xC2, 0x10, 0x07,
	0x05, 0xC5, 0x87, 0x90,
	0x12, 0x31, 0x23, 0x47,
	0x84, 0x87, 0x90, 0x37,
	0x38, 0x0C, 0x00, 0x18,
	0x00, 0x00, 0x00, 0x0C,
	0x00, 0x18, 0x00, 0x00,
	0x00, 0x88, 0x87, 0x57,
	0x53, 0x18, 0x88, 0x88,
	0x88, 0x88, 0x13, 0x88,
	0x88, 0x86, 0x46, 0x42,
	0x08, 0x88, 0x88, 0x88,
	0x88, 0x02, 0x88, 0x00,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00,

};

static char st7703_720p_video_on_cmd15[] = {
	0x3E, 0x00, 0x39, 0xC0,
	0xEA, 0x02, 0x21, 0x00,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x88, 0x80, 0x24,
	0x60, 0x28, 0x88, 0x88,
	0x88, 0x88, 0x64, 0x88,
	0x88, 0x81, 0x35, 0x71,
	0x38, 0x88, 0x88, 0x88,
	0x88, 0x75, 0x88, 0x23,
	0x10, 0x00, 0x00, 0x08,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x40,
	0x87, 0x90, 0x00, 0x00,
	0x00, 0x00, 0xFF, 0xFF,
};

static char st7703_720p_video_on_cmd16[] = {
	0x23, 0x00, 0x39, 0xC0,
	0xE0, 0x00, 0x1A, 0x23,
	0x27, 0x28, 0x3A, 0x50,
	0x4E, 0x07, 0x0B, 0x0C,
	0x0F, 0x11, 0x10, 0x13,
	0x19, 0x1A, 0x00, 0x1A,
	0x23, 0x27, 0x28, 0x3A,
	0x50, 0x4E, 0x07, 0x0B,
	0x0C, 0x0F, 0x11, 0x10,
	0x13, 0x19, 0x1A, 0xFF,
};

static char st7703_720p_video_on_cmd17[] = {
	0x11, 0x00, 0x05, 0x80
};

static char st7703_720p_video_on_cmd18[] = {
	0x29, 0x00, 0x05, 0x80
};

static struct mipi_dsi_cmd st7703_720p_video_on_command[] = {
	{0x8, st7703_720p_video_on_cmd0, 0x00},
	{0x20, st7703_720p_video_on_cmd1, 0x00},
	{0xc, st7703_720p_video_on_cmd2, 0x00},
	{0x10, st7703_720p_video_on_cmd3, 0x00},
	{0x10, st7703_720p_video_on_cmd4, 0x00},
	{0x8, st7703_720p_video_on_cmd5, 0x00},
	{0x8, st7703_720p_video_on_cmd6, 0x00},
	{0x8, st7703_720p_video_on_cmd7, 0x00},
	{0x8, st7703_720p_video_on_cmd8, 0x00},
	{0x14, st7703_720p_video_on_cmd9, 0x00},
	{0x14, st7703_720p_video_on_cmd10, 0x00},
	{0x8, st7703_720p_video_on_cmd11, 0x00},
	{0x8, st7703_720p_video_on_cmd12, 0x00},
	{0x8, st7703_720p_video_on_cmd13, 0x00},
	{0x44, st7703_720p_video_on_cmd14, 0x00},
	{0x44, st7703_720p_video_on_cmd15, 0x00},
	{0x28, st7703_720p_video_on_cmd16, 0x00},
	{0x4, st7703_720p_video_on_cmd17, 0xFA},
	{0x4, st7703_720p_video_on_cmd18, 0x32}
};

#define ST7703_720P_VIDEO_ON_COMMAND 19


static char st7703_720p_videooff_cmd0[] = {
	0x28, 0x00, 0x05, 0x80
};	
static char st7703_720p_videooff_cmd1[] = {
	0x10, 0x00, 0x05, 0x80
};	
static struct mipi_dsi_cmd st7703_720p_video_off_command[] = {
	{0x4, st7703_720p_videooff_cmd0, 0xC8},
	{0x4, st7703_720p_videooff_cmd1, 0x96},
};

#define ST7703_720P_VIDEO_OFF_COMMAND 2


static struct command_state st7703_720p_video_state = {
	0, 1
};

/*---------------------------------------------------------------------------*/
/* Command mode panel information                                            */
/*---------------------------------------------------------------------------*/
static struct commandpanel_info st7703_720p_video_command_panel = {
	0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
};

/*---------------------------------------------------------------------------*/
/* Video mode panel information                                              */
/*---------------------------------------------------------------------------*/
static struct videopanel_info st7703_720p_video_video_panel = {
	1, 0, 0, 0, 1, 1, 2, 0, 0x9
};

/*---------------------------------------------------------------------------*/
/* Lane configuration                                                        */
/*---------------------------------------------------------------------------*/
static struct lane_configuration st7703_720p_video_lane_config = {
	4, 0, 1, 1, 1, 1, 0
};

/*---------------------------------------------------------------------------*/
/* Panel timing                                                              */
/*---------------------------------------------------------------------------*/
static const uint32_t st7703_720p_video_timings[] = {
	0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
    0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
	0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
	0x1e, 0x1c ,0x04 ,0x06 ,0x02 ,0x03 ,0x04 ,0xa0,
	0x1e ,0x0e ,0x04 ,0x05 ,0x02 ,0x03 ,0x04 ,0xa0
};

static struct panel_timing st7703_720p_video_timing_info = {
	0, 4, 0x04, 0x1b
};

/*---------------------------------------------------------------------------*/
/* Panel reset sequence                                                      */
/*---------------------------------------------------------------------------*/
static struct panel_reset_sequence st7703_720p_video_reset_seq = {
	{1, 0, 1, }, {20, 2, 60, }, 2
};

/*---------------------------------------------------------------------------*/
/* Backlight setting                                                         */
/*---------------------------------------------------------------------------*/
static struct backlight st7703_720p_video_backlight = {
	1, 1, 4095, 100, 0, "PMIC_8941"
};

#endif /*_PANEL_ST7703_720P_VIDEO_H_*/
```

加新的面板參數

*bootable/bootloader/lk/target/msm8953/oem_panel.c*

```xml-dtd
diff --git a/bootable/bootloader/lk/target/msm8953/oem_panel.c b/bootable/bootloader/lk/target/msm8953/oem_panel.c
index c88f3f0..33157ff 100644
--- a/bootable/bootloader/lk/target/msm8953/oem_panel.c
+++ b/bootable/bootloader/lk/target/msm8953/oem_panel.c
@@ -55,13 +55,14 @@
 #include "include/panel_ili9881c_720p_video.h"
 #include "include/panel_hx8394f_720p_video.h"
 #include "include/panel_gt101wu_1920p_video.h"
-
+#include "include/panel_st7703_720p_video.h"
 /*---------------------------------------------------------------------------*/
 /* static panel selection variable                                           */
 /*---------------------------------------------------------------------------*/
 static uint32_t auto_pan_loop = 0;
 
 enum {
+	ST7703_720P_VIDEO_PANEL,
 	GT101WU_1920P_VIDEO_PANEL,
 	HX8394F_720P_VIDEO_PANEL,
 	ILI9881C_720P_VIDEO_PANEL,
@@ -80,6 +81,7 @@ enum {
  * Any panel in this list can be selected using fastboot oem command.
  */
 static struct panel_list supp_panels[] = {
+	{"st7703_720p_video", ST7703_720P_VIDEO_PANEL},
 	{"gt101wu_1920p_video", GT101WU_1920P_VIDEO_PANEL},
 	{"hx8394f_720p_video", HX8394F_720P_VIDEO_PANEL},
 	{"ili9881c_720p_video", ILI9881C_720P_VIDEO_PANEL},
@@ -127,6 +129,33 @@ static int init_panel_data(struct panel_struct *panelstruct,
 	int pan_type = PANEL_TYPE_DSI;
 
 	switch (panel_id) {
+	case ST7703_720P_VIDEO_PANEL:
+		panelstruct->paneldata    = &st7703_720p_video_panel_data;
+		panelstruct->paneldata->panel_with_enable_gpio = 0;
+		panelstruct->panelres     = &st7703_720p_video_panel_res;
+		panelstruct->color        = &st7703_720p_video_color;
+		panelstruct->videopanel   = &st7703_720p_video_video_panel;
+		panelstruct->commandpanel = &st7703_720p_video_command_panel;
+		panelstruct->state        = &st7703_720p_video_state;
+		panelstruct->laneconfig   = &st7703_720p_video_lane_config;
+		panelstruct->paneltiminginfo
+			= &st7703_720p_video_timing_info,
+		panelstruct->panelresetseq
+			= &st7703_720p_video_reset_seq,
+		panelstruct->backlightinfo = &st7703_720p_video_backlight;
+		pinfo->mipi.panel_on_cmds
+			= st7703_720p_video_on_command,
+		pinfo->mipi.num_of_panel_on_cmds
+			= ST7703_720P_VIDEO_ON_COMMAND,
+		pinfo->mipi.panel_off_cmds
+			= st7703_720p_video_off_command,
+		pinfo->mipi.num_of_panel_off_cmds
+			= ST7703_720P_VIDEO_OFF_COMMAND,
+		memcpy(phy_db->timing,
+			st7703_720p_video_timings,
+			MAX_TIMING_CONFIG * sizeof(uint32_t));
+		pinfo->mipi.signature = 0xFFFF;//ST7703_720P_VIDEO_SIGNATURE;	
+		break;
 	case GT101WU_1920P_VIDEO_PANEL:
 		panelstruct->paneldata    = &gt101wu_1920p_video_panel_data;
 		panelstruct->paneldata->panel_with_enable_gpio = 0;
@@ -469,7 +498,7 @@ int oem_panel_select(const char *panel_name, struct panel_struct *panelstruct,
 	case HW_PLATFORM_MTP:
 	case HW_PLATFORM_SURF:
 	case HW_PLATFORM_RCM:
-		panel_id = GT101WU_1920P_VIDEO_PANEL;
+		panel_id = ST7703_720P_VIDEO_PANEL;
 		if(auto_pan_loop == 0)
 		{
 //			panel_id = ILI9881C_720P_VIDEO_PANEL;
```

以上lk的面板就點亮了

## 5) kernel修改

把dsi-panel-st7703-720p-video.dtsi移到kernel/msm-4.9/arch/arm64/boot/dts/qcom/目錄下
根據Qutecl官方文件和LCM規格修改

```xml-dtd
--- device/qcom/common/display/tools/dsi-panel-st7703-720p-video.dtsi   2020-08-20 09:18:45.886411475 +0800
+++ kernel/msm-4.9/arch/arm64/boot/dts/qcom/dsi-panel-st7703-720p-video.dtsi    2020-08-10 17:07:17.965514437 +0800
@@ -74,6 +74,12 @@
                qcom,mdss-dsi-lane-2-state;
                qcom,mdss-dsi-lane-3-state;
                qcom,mdss-dsi-panel-timings = [87 1c 12 00 42 44 18 20 17 03 04 00];
+               qcom,mdss-dsi-panel-timings-phy-v2 = [
+                           1e 1c 04 06 02 03 04 a0
+                                   1e 1c 04 06 02 03 04 a0
+                                   1e 1c 04 06 02 03 04 a0
+                                   1e 1c 04 06 02 03 04 a0
+                                   1e 0e 04 05 02 03 04 a0];
                qcom,mdss-dsi-t-clk-post = <0x04>;
                qcom,mdss-dsi-t-clk-pre = <0x1b>;
                qcom,mdss-dsi-bl-min-level = <1>;
@@ -81,6 +87,11 @@
                qcom,mdss-dsi-dma-trigger = "trigger_sw";
                qcom,mdss-dsi-mdp-trigger = "none";
                qcom,mdss-dsi-bl-pmic-control-type = "bl_ctrl_pwm";
-               qcom,mdss-dsi-reset-sequence = <1 20>, <0 2>, <1 60>;
+               qcom,mdss-dsi-reset-sequence = <1 20>, <0 10>, <1 120>;
+               qcom,mdss-pan-physical-width-dimension = <70>;
+               qcom,mdss-pan-physical-height-dimension = <144>;
+               qcom,mdss-dsi-post-init-delay = <1>;
+               qcom,mdss-dsi-force-clock-lane-hs;
+               qcom,cont-splash-enabled;
        };
 };

```

把面板參數整合至dsi透過device tree

```xml-dtd
diff --git a/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-mdss-panels.dtsi b/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-mdss-panels.dtsi
index 4dfbc2e..08951f2 100755
--- a/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-mdss-panels.dtsi
+++ b/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-mdss-panels.dtsi
@@ -33,6 +33,7 @@
 #include "dsi-panel-hx8394f-720p-video.dtsi"
 #include "dsi-panel-hx8394f-720p-dsi1-video.dtsi"
 #include "dsi-panel-gt101wu-1920p-video.dtsi"
+#include "dsi-panel-st7703-720p-video.dtsi"
 
 &soc {
 	dsi_panel_pwr_supply: dsi_panel_pwr_supply {
@@ -254,3 +255,12 @@
         24 1f 08 09 05 03 04 a0     /*Data 3*/
         24 1c 08 09 05 03 04 a0];   /*CLK lane*/
 };
+
+&dsi_st7703_720p_video {
+    qcom,mdss-dsi-panel-timings-phy-v2 = [
+        1e 1c 04 06 02 03 04 a0		/*Data 0*/
+	    1e 1c 04 06 02 03 04 a0		/*Data 1*/
+	    1e 1c 04 06 02 03 04 a0		/*Data 2*/
+	    1e 1c 04 06 02 03 04 a0		/*Data 3*/
+	    1e 0e 04 05 02 03 04 a0];   /*CLK lane*/
+};
diff --git a/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-nopmi-panel-camera.dtsi b/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-nopmi-panel-camera.dtsi
index 4fc2b95..c42332d 100755
--- a/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-nopmi-panel-camera.dtsi
+++ b/kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-nopmi-panel-camera.dtsi
@@ -58,7 +58,7 @@
 };
 
 &mdss_dsi0 {
-	qcom,dsi-pref-prim-pan = <&dsi_gt101wu_1920p_video>;
+	qcom,dsi-pref-prim-pan = <&dsi_st7703_720p_video>;
 	pinctrl-names = "mdss_default", "mdss_sleep";
 	pinctrl-0 = <&mdss_dsi_active &mdss_te_active>;
 	pinctrl-1 = <&mdss_dsi_suspend &mdss_te_suspend>;

    qcom,platform-te-gpio = <&tlmm 24 0>;
    qcom,platform-reset-gpio = <&tlmm 61 0>;
@@ -139,6 +139,14 @@
 	qcom,mdss-dsi-pwm-gpio = <&pm8953_mpps 4 0>;
 };
 
+&dsi_st7703_720p_video {
+	qcom,panel-supply-entries = <&dsi_panel_pwr_supply>;
+	qcom,mdss-dsi-bl-pmic-control-type = "bl_ctrl_pwm";
+	qcom,mdss-dsi-bl-pmic-pwm-frequency = <100>;
+	qcom,mdss-dsi-bl-pmic-bank-select = <0>;
+	qcom,mdss-dsi-pwm-gpio = <&pm8953_mpps 4 0>;
+};
+
 &soc {
 //rid:add dsi1 bridge for dual mipi dsi
     qcom,dsi1_bridge {
@@ -158,4 +166,4 @@
         pinctrl-0 = <&gpio_clk_default>;
         pinctrl-1 = <&gpio_clk_sleep>;
     };
-};
\ No newline at end of file
+};
```

以上kernel就可點亮LCM

![image-20200820102034102]({{site.baseurl}}/img/image-20200820102034102.png)