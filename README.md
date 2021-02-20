/******************************************************************************
* Assembly EECE 244 Lab 3 - BCD Integer Input Display
*This program will accept two Valid BCD values and add them up to 18, if
*the sum of the two is greater then 9 then a decimal point will display
*in place of the 10th digit.
*
* Andy Nguyen, 02/10/2021 Email: Nguye218@wwu.edu
******************************************************************************/

        .syntax unified                           //Defined Syntax
        .cpu cortex-m4
        .fpu fpv4-sp-d16

        .globl main                     //Makes main() global



       //Defining BITS
          .equ BIT0, 0x01
          .equ BIT1, 0x01<<1
          .equ BIT2, 0x01<<2
          .equ BIT3, 0x01<<3
          .equ BIT4, 0x01<<4
          .equ BIT5, 0x01<<5
          .equ BIT6, 0x01<<6
          .equ BIT7, 0x01<<7
          .equ BIT8, 0x01<<8
          .equ BIT9, 0x01<<9
          .equ BIT10, 0x01<<10
          .equ BIT11, 0x01<<11
          .equ BIT12, 0x01<<12
          .equ BIT13, 0x01<<13
          .equ BIT14, 0x01<<14
          .equ BIT15, 0x01<<15
          .equ BIT16, 0x01<<16
          .equ BIT17, 0x01<<17
          .equ BIT18, 0x01<<18
          .equ BIT19, 0x01<<19
          .equ BIT20, 0x01<<20
          .equ BITFF, 0xFFFFFFFF
          .equ BYTE_CLEAR, 0x000000FF
          .equ MUX_PCR, 0x01<<8
        //Defining Port Addresses
          .equ PORTB_PCR0, 0x4004A000   // Define Pin Control Register Port B
          .equ PORTC_PCR0, 0x4004B000   // Define Pin Control Register Port C
          .equ PORTD_PCR0, 0x4004C000   // Define Pin Control Register Port D

        //offsets
          .equ PDOR, 4*0     //Output Register
          .equ PDDR, 4*5     //Direction Register
          .equ PDIR, 4*4     //Input Register

         //PCR offsets
          .equ PCR0, 4*0
          .equ PCR1, 4*1
          .equ PCR2, 4*2
          .equ PCR3, 4*3
          .equ PCR4, 4*4
          .equ PCR5, 4*5
          .equ PCR6, 4*6
          .equ PCR7, 4*7
          .equ PCR8, 4*8
          .equ PCR9, 4*9
          .equ PCR16, 4*16
          .equ PCR18, 4*18
          .equ PCR19, 4*19

        //Clock
          .equ SIM_SCGC5, 0x40048038    // Initialize the clock to register 5

        //Setting up ports for Clcok MUX
          .equ SIM_PORTB, 0x01<<10      // Defining ports B
          .equ SIM_PORTC, 0x01<<11      // Defining ports C
          .equ SIM_PORTD, 0x01<<12      // Defining ports D

        //Data Direction Register for Port B and D
          .equ GPIOD_PDDR, 0x400FF0D4   // Define Port Data Direction Register D
          .equ GPIOB_PDDR, 0x400FF054   // Define Port Data Direction Register B

       //Data Input Register for Port C
          .equ GPIOC_PDIR, 0x400FF090   // Define Port Data Input Register C

          .equ GPIOC, 0x400FF090   // Define Port Data Input Register C
          .equ GPIOD, 0x400FF0C0
          .equ GPIOB, 0x400FF040


        //Pull enable pull switch enable MUX
          .equ PORT_MUX, 0x100
          .equ PORT_PE_ON, 0x01<<1
          .equ PORT_PS_UP, 0x01

          .equ CLEAR, 0x00000000

          .equ DASH, 0x1C
          .equ DEC, 0x00
          .equ DASH_B, 0x00090003
          .equ DEC_OFF, 0x01

          .section .text                            // The following is program code

main:

        bl IOSheildInit

//This portion Clears registers R1-R6
        and r1, #CLEAR
        and r2, #CLEAR
        and r3, #CLEAR
        and r4, #CLEAR
        and r5, #CLEAR
        and r6, #CLEAR

//Checks if the switches are the same from lastSw
SwitchCheck:

        movw r2, #:lower16:GPIOC_PDIR    //Read GPOIC_PDIR
        movt r2, #:upper16:GPIOC_PDIR
        ldr r4, [r2]                    //load the contents of
        ldr r3, =LastSw                 //load LastSw conents
        ldr r5, [r3]                    //load r5 witht the contents in LastSw
        cmp r5, r4                      //comparison of last swith and GIOC

//loop break
beq SwitchCheck

        str r4, [r3]                     //Updates LastSw value

//turns off the Decimal point initically
        movw r1, #:lower16:GPIOB         // Read GPIOB
        movt r1, #:upper16:GPIOB
        ldr r2, [r1]                     //loads the contents of GPIOB into r2
        mov r3, #DEC_OFF                 //moves DEC_OFF into r3
        bfi r2, r3, #19, #1
        str r2, [r1]                     //Writes DEC_OFF into GPIOB

        bl SwArrayRead                  //Calls the SwArrayRead

//Clears r1-r4 registers for BFI
        and r1, #CLEAR
        and r2, #CLEAR
        and r3, #CLEAR
        and r4, #CLEAR

        bfi r1, r0, #0, #4  //Right input switch Value
        lsr r0, #4          //Push all the way to the right 4 bits for next BFI
        bfi r2, r0, #0, #4  //Left input switch Value

        subs r4, r1, #10    //subtract Right input value by 10
        bpl Invalid_Input   //Check if Value is Invalid
        subs r4, r2, #10    //subtract Left input value by 10
        bpl Invalid_Input   //Check if Value is Invalid

        b Valid_Input       //Branch if Valid Input

Invalid_Input:

        movw r1, #:lower16:GPIOD
        movt r1, #:upper16:GPIOD      //loads GPIOD
        ldr r2, =DASH                 //loads DASH
        str r2, [r1]                  //puts DASH into GPIOD

        movw r1, #:lower16:GPIOB
        movt r1, #:upper16:GPIOB      // next two lines are the same as above
        ldr r2, =DASH_B
        str r2, [r1]                  //writes Dash for the invalid input

        b SwitchCheck                 //breaks with loop

Valid_Input:


        add r4, r2, r1                //adds input values
        mov r6, r4                    //moves the reslut of addition into r6
        subs r4, #10                  //subtract 10 for conditional branch

        bmi LessThenTen               //branch if neg


//this line all the way to the sub is the
        movw r1, #:lower16:GPIOB
        movt r1, #:upper16:GPIOB
        ldr r2, [r1]                  //loads the contents of GPIOB into r2
        mov r3, #DEC                  //moves DEC into r3
        bfi r2, r3, #19, #1           //BFI r2 and r3 to enable DEC
        str r2, [r1]                  //wreite the DEC point
        sub r6, #10                   //subtract 10 from r6 for lsb

LessThenTen:

        mov r0, r6                    //moves lsb into r0 for the LEDWrite comand

        bl LEDWrite                    //calls the LEDWrite Subroutine

        b SwitchCheck                  //loops back to SwitchCheck





/**************************************************************************
* void IOShieldInit(void); – This subroutine initializes the ports.
* The sevenseg display, and the switch.
*
* Params: An 8-bit integer passed through in r0
* Returns: none
* MCU: K22F
* Andy Nguyen, 02/10/2021
**************************************************************************/

IOSheildInit:

      push {lr}


      //Clock Enable for PORTS//
         movw r0, #:lower16:SIM_SCGC5
         movt r0, #:upper16:SIM_SCGC5
         ldr r1, [r0]                               //Reads SCGC5 from r0
         orr r1, #(SIM_PORTB|SIM_PORTC|SIM_PORTD)   //Modify and set PORT's B,C,D bit
         str r1, [r0]                               //Stores and write SCG5


      //Port-D outputs D2,D3,D4
         movw r2, #:lower16:PORTD_PCR0              //gets port D address
         movt r2, #:upper16:PORTD_PCR0
         mov r0, #MUX_PCR                           //writes BIT8 MUX tp PCR
         str r0, [r2, #PCR2]
         str r0, [r2, #PCR3]
         str r0, [r2, #PCR4]


         movw r2, #:lower16:GPIOD_PDDR              //gets GIOD base address
         movt r2, #:upper16:GPIOD_PDDR
         mov r0, #(BIT2|BIT3|BIT4)                  //Makes bits 2-4 Outputs
         str r0, [r2,#PDDR]

         movw r2, #:lower16:GPIOD_PDDR
         movt r2, #:upper16:GPIOD_PDDR
         ldr r0, [r2,#PDOR]                         //Read PDOR
         orr r0, #(BIT2|BIT3|BIT4)                  //Modify - set bit(s) 2,3,4
         str r0, [r2,#PDOR]                         //Write PDOR


      //Port-B Outputs
         movw r2, #:lower16:PORTB_PCR0                //gets port B address
         movt r2, #:upper16:PORTB_PCR0
         mov r0, #BIT8                                //writes BIT8 MUX to PCR
         str r0, [r2, #PCR0]
         str r0, [r2, #PCR1]
         str r0, [r2, #PCR16]
         str r0, [r2, #PCR18]
         str r0, [r2, #PCR19]

         movw r2, #:lower16:GPIOB_PDDR
         movt r2, #:upper16:GPIOB_PDDR
         mov r0, #BIT1                                 //Makes bit 1 and output
         str r0, [r2,#PDDR]                            //Write PDOR

         movw r2, #:lower16:GPIOB_PDDR
         movt r2, #:upper16:GPIOB_PDDR
         movw r0, #:lower16: #(BIT0|BIT1)              //Mask For 5 outputs
         movt r0, #:upper16: #(BIT16|BIT18|BIT19)      //LED - D3-D7
         str r0, [r2,#PDOR]                            //Write PDOR



    //Port-C Input
         movw r2, #:lower16:PORTC_PCR0
         movt r2, #:upper16:PORTC_PCR0
         movw r0, #(PORT_PE_ON|PORT_MUX|PORT_PS_UP)   //Pull enable pull switch enable
         str r0, [r2, #PCR2]
         str r0, [r2, #PCR3]
         str r0, [r2, #PCR4]
         str r0, [r2, #PCR5]
         str r0, [r2, #PCR6]
         str r0, [r2, #PCR7]
         str r0, [r2, #PCR8]
         str r0, [r2, #PCR9]


      pop {pc}

/**************************************************************************
* INT8U SwArrayRead(void)–This subroutine reads the input from the switches
* and store the value as an 8bits integer into r0.
*
* Params: none
* Returns: An 8-bit integer
* MCU: K22F
* Andy Nguyen, 02/10/2021
**************************************************************************/

 SwArrayRead:

        push {lr}

         movw r0, #:lower16:GPIOC_PDIR    //Read GPOIC_PDIR
         movt r0, #:upper16:GPIOC_PDIR
         ldr r1, [r0]
         lsr r1, #2                      //Logical shit Right 2
         mov r0, r1                      //moves r1 into r0

        pop {pc}


/**************************************************************************
* void LEDWrite(INT8U lbit) – This subroutine gets the 8 bit integer
* and outputs it on the seven segment display.
*
* Params: An 8-bit integer passed through in r0.
* Returns: none
* MCU: K22F
* Andy Nguyen, 02/10/2021
**************************************************************************/

LEDWrite:

           push {lr}

         ldr r1, =SevenSegTable
         ldr r5, [r1, r0]

         eor r5, #BITFF                   //xor to invert the registers

      // SW0-SW2
         movw r1, #:lower16:GPIOD         // Read GPIOD
         movt r1, #:upper16:GPIOD

    	 ldr r8, [r1]
   // 	 lsr r5, #2 (void)
    	 bfi r8, r5, #2,#3                // 4 bits from r5 put into r8 at pos 2
    	 lsr r5, #3
    	 str r8, [r1]                     // write the output into r8

     // SW3-SW4
         movw r1, #:lower16:GPIOB         // Read GPIOB
         movt r1, #:upper16:GPIOB
    	 ldr r8, [r1]
    	 bfi r8, r5, #0,#2                // 2 bits from r5 put into r8 at pos 8
     	 lsr r5, #2
    	 str r8, [r1]                     // write the output into r8

      //SW5

    	 ldr r8, [r1]
    	 bfi r8, r5, #16,#1               // 1 bit from r5  put into r8 at pos 16
       	 lsr r5, #1
    	 str r8, [r1]                     // write the output into r8

       //SW6, SW7

    	 ldr r8, [r1]
    	 bfi r8, r5, #18,#1               // 2 bits from r5 put into r8 at pos 18
    	 str r8, [r1]                     // write the output into r8

         pop {pc}




.section .rodata
/* place defined constants here */
SevenSegTable:  .byte 0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x6D, 0x7D, 0x07, 0x7F, 0x6F // the LUT for the output of 7 segment display

.section .bss


/* place RAM variables here */
.comm LastSw, 4

