//Header file-------------------------------------------------------------
#include <xc.h>
#define _XTAL_FREQ 4000000//Definition of register and register bytes.

//See pic16f1827.h in compilator’s include catalog
//C:\Program Files\Microchip\xc8\Vx.xx\include
//Configuration Bits-------------------------------------------------------
//Ses C:\Program Files\Microchip\xc8\Vx.xx\docs\pic_chipinfo
//Oscillator Intern, Watchdog timer disabled, Power-Up timer enabled
//Brownout Reset Disable, Low-Voltage Programming Disabled, Unprotect memory,
//MasterClear Input Enabled, Debug disabled,FCM/IESO disabled

#pragma config CPD=OFF, BOREN=OFF, IESO=OFF, FOSC=INTOSC, FCMEN=OFF, MCLRE=ON,\
 WDTE=OFF, CP=OFF, PWRTE=ON, CLKOUTEN=ON //Config Word 1
#pragma config PLLEN=OFF, WRT=OFF, STVREN=ON, BORV=LO, LVP=OFF //Config Word 2

//Functions prototypes---------------------------------------------------------
void floor_update(void);
void speed_control(void);
void init(void);
char AD_omv(void);
void temperature_check(void);

//Global variables-----------------------------------------------------------
char AD_in;

//Main program----------------------------------------------------------------
char AD_in;
void main()
{
   init();
   while(1)
   {
     floor_update(); //Updates the 7-segment with current position
     speed_control(); //Control speed info + soft launch
     AD_in=AD_omv(); //Converts analog signal to digital
     temperature_check(); //Controls if temperature <30C
   }
}

//Function----------------------------------------------------------------
void init()
{
OSCCON=0b01101000; //Intern clock 4 MHz
ANSELA=0b00010000; //PORTA all bytes digital except byte 4 which is analog
ANSELB=0b00000000; //PORTB all bytes digital
TRISB=0; //PORTB all bytes outputsignal
TRISA=0b10110011; //PORTA byte 7,5,4,1,0 input, rest output
LATB = 0b00000000; //Reset ports from start
LATA = 0b00000000; //Reset ports from start
CCP3CON=0b00001100; //CCP3 in PWM-mode
CCPTMRS=0b01001010; //CCP3 uses Timer2
PR2=254;
T2CON=0b00000110; //Timer2 On, Prescaler=1 gives fPWM = 250Hz
ADCON1=0b01000000; //Adjusted to the left, AD-clock=Fosc/4, Vref VDD and VSS
}
void floor_update(){
   if(!PORTAbits.RA1 && !PORTAbits.RA0){        //MSB=0 LSB=0
      LATB = 0b00000110;                        //Level 1
   }
   else if(!PORTAbits.RA1 && PORTAbits.RA0){    //MSB=0 LSB=1
      LATB = 0b01011011;                        //Level 2
   }
   else if(PORTAbits.RA1 && !PORTAbits.RA0){    //MSB=1 LSB=0
      LATB = 0b01001111;                        //Level 3
   }
   else if(PORTAbits.RA1 && PORTAbits.RA0){     //MSB=1 LSB=1
      LATB = 0b01100110;                        //Level 4
   }
}
void speed_control(){
    if(PORTAbits.RA7 && (CCPR3L < 256)){
      __delay_ms(10);               //Delay 10ms
      CCPR3L++;                     //Duty cycle increase 0-100%
    }
    if(!PORTAbits.RA7){             //LE byte inactive
         CCPR3L = 0;                //Reset speed to 0%
    }
}
char AD_omv(){
    ADCON0=0b00010001;          //choose AD-Canal 4
    __delay_us(5);              //Delay 5us
    ADCON0bits.GO=1;            //AD-converter starts
    while(ADCON0bits.GO){}      //waits until AD is finished
    return ADRESH;              //Return 8 MSB av AD-converter
}
void temperature_check(){
    if(AD_in>=0x83){            //if AD_in >30C
    LATAbits.LATA2=1;           //Activate alarm
    }else{
    LATAbits.LATA2=0;           //Deactivate alarm
    }
}

*****END CODE*****
