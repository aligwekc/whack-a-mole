;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;NAME: ALIGWEKWE CHIAMAKA
;DATE: 06-12-2019
;CLASS: ENSE352 PROJECT
;PROJECT: WHACK_A_MOLE
;REQUIRE: R3 contains all LED register address 
;		  R6 contains all button register address
;         R8 contains the time delay 
;         R12 contains the new seed generated 
;PROMISE: R4 is where the LED adresses are store in order to turn them off or on
;	      R7 is where the button adresses are store in order to connect them to the LED and check if they are pressed
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; Directives
            PRESERVE8
            THUMB                   
;;; Equates
INITIAL_MSP    EQU        0x20001000    ; Initial Main Stack Pointer Value

;The board LEDS are on port C bits 8 and 9
;The board switches which would be used to control the LED are on poers A and B, 
;Therefore we have to initialise all ports A, B and C to configure and enable the switches and LED
;PORT A GPIO - Base Addr: 0x40010800
GPIOA_CRL    EQU        0x40010800    ; (0x00) Port Configuration Register Low for Px7 -> Px0
GPIOA_CRH    EQU        0x40010804    ; (0x04) Port Configuration Register high for Px15 -> Px8
GPIOA_IDR    EQU        0x40010808    ; (0x08) Port Input Data Register
GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
GPIOA_BSRR   EQU        0x40010810    ; (0x10) Port Bit Set/Reset Register
GPIOA_BRR    EQU        0x40010814    ; (0x14) Port Bit Reset Register
GPIOA_LCKR   EQU        0x40010818    ; (0x18) Port Configuration Lock Register
	

;PORT B GPIO - Base Addr: 0x40010C00
GPIOB_CRL	EQU		0x40010C00	; (0x00) Port Configuration Register for Px7 -> Px0
GPIOB_CRH	EQU		0x40010C04	; (0x04) Port Configuration Register for Px15 -> Px8
GPIOB_IDR	EQU		0x40010C08	; (0x08) Port Input Data Register
GPIOB_ODR	EQU		0x40010C0C	; (0x0C) Port Output Data Register
GPIOB_BSRR	EQU		0x40010C10	; (0x10) Port Bit Set/Reset Register
GPIOB_BRR	EQU		0x40010C14	; (0x14) Port Bit Reset Register
GPIOB_LCKR	EQU		0x40010C18	; (0x18) Port Configuration Lock Register
	
;PORT C GPIO - Base Addr: 0x40011000
GPIOC_CRL	EQU		0x40011000	; (0x00) Port Configuration Register for Px7 -> Px0
GPIOC_CRH	EQU		0x40011004	; (0x04) Port Configuration Register for Px15 -> Px8
GPIOC_IDR	EQU		0x40011008	; (0x08) Port Input Data Register
GPIOC_ODR	EQU		0x4001100C	; (0x0C) Port Output Data Register
GPIOC_BSRR	EQU		0x40011010	; (0x10) Port Bit Set/Reset Register
GPIOC_BRR	EQU		0x40011014	; (0x14) Port Bit Reset Register
GPIOC_LCKR	EQU		0x40011018	; (0x18) Port Configuration Lock Register
	
	
;Registers for configuring and enabling the clocks used in the game 
;RCC Registers - Base Addr: 0x40021000
RCC_CR		EQU		0x40021000	; Clock Control Register
RCC_CFGR	EQU		0x40021004	; Clock Configuration Register
RCC_CIR		EQU		0x40021008	; Clock Interrupt Register
RCC_APB2RSTR	EQU	0x4002100C	; APB2 Peripheral Reset Register
RCC_APB1RSTR	EQU	0x40021010	; APB1 Peripheral Reset Register
RCC_AHBENR	EQU		0x40021014	; AHB Peripheral Clock Enable Register
RCC_APB2ENR	EQU		0x40021018	; APB2 Peripheral Clock Enable Register  -- Used
RCC_APB1ENR	EQU		0x4002101C	; APB1 Peripheral Clock Enable Register
RCC_BDCR	EQU		0x40021020	; Backup Domain Control Register
RCC_CSR		EQU		0x40021024	; Control/Status Register
RCC_CFGR2	EQU		0x4002102C	; Clock Configuration Register 2


;Delay times 
delay_time    EQU       800000        
time_delay   EQU        50000
game_delay   EQU        500000
mole_delay    EQU       500000

; Vector Table Mapped to Address 0 at Reset
            AREA    RESET, Data, READONLY
            EXPORT  __Vectors

__Vectors    DCD        INITIAL_MSP            ; stack pointer value when stack is empty
            DCD        Reset_Handler        ; reset vector
            
            AREA    MYCODE, CODE, READONLY
            EXPORT    Reset_Handler
            ENTRY

Reset_Handler        PROC

    BL GPIO_ClockInit
    BL GPIO_init
    
    BL turn_off_R
    LDR R5, = 0
    

    
;this subroutines turns off the LED lights
turn_off_R     
    LDR R3, = GPIOA_ODR            ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
    LDR R4, [R3]
    ORR R4, #0xFFFFFFFF        ;0001 1110 to make the LEDs disable
    STR R4, [R3]
    LDR R8, = time_delay
	 B LED4
	
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;LED subroutines;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
LED4 ;turns on the 4th LED
	LDR R3,= GPIOA_ODR                          ;  (0x0C) Port Output Data Register        
	LDR R4, [R3]                                ; loading the LED digits into another register to make changes to it 
	AND R4, #0xFFFFEFFF                         ; turns on the LED which is active low
	STR R4, [R3]                                ; store the changes back in the original register, this turns on the light
	BL green_button                             ;this will branch but also link, the green button associated with the LED so as to return back to this register 
	BL blue_button                              ;this will branch but also link, the blue button associated with the LED  
	BL black_button                             ;this will branch but also link, the black button associated with the LED 
	BL red_button                               ;this will branch but also link, the red button associated with the LED
	B delay4                                    ; to time how long the LED would be on for, before turning it off 

LED3
	LDR R3,= GPIOA_ODR                      ;  (0x0C) Port Output Data Register        
	LDR R4, [R3]                             ; loading the LED digits into another register to make changes to it 
	AND R4, #0xFFFFF7FF                        ; turns on the LED which is active low
	STR R4, [R3]                              ; store the changes back in the original register, this turns on the light
	BL green_button                           ;this will branch but also link, the green button associated with the LED so as to return back to this register 
	BL blue_button
	BL black_button
	BL red_button
	B delay3    ; this would count how long the LEd is on for, when the time put in register 8 is equal to zero the next LED would come on while LED3 goes off

LED2
	LDR R3, = GPIOA_ODR                           ;  (0x0C) Port Output Data Register 
	LDR R4, [R3]                                    ; loading the LED digits into another register to make changes to it  
	AND R4, #0xFFFFFBFF                             ; turns on the LED which is active low
	STR R4, [R3]                                    ; store the changes back in the original register, this turns on the light
	BL green_button                                 ;this will branch but also link, the green button associated with the LED so as to return back to this register 
	BL blue_button                               
	BL black_button
	BL red_button
	B delay2                ; this would count how long the LEd is on for, when the time put in register 8 is equal to zero the next LED would come on while LED2 goes off 
	
LED1 
	LDR R3,= GPIOA_ODR
	LDR R4, [R3]
	AND R4, #0xFFFFFDFF
	STR R4, [R3]
	BL green_button
	BL blue_button
	BL black_button
	BL red_button
	B delay1           ; this would count how long the LEd is on for, when the time put in register 8 is equal to zero the next LED would come on while LED goes off
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;END of LED subroutine;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;DELAY SUBROUTINES FOR EACH LED;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;subroutines that tells the LED to turn off when register 8 is equal to zero R8 = time_delay
delay4
    SUB R8, R8, #1   ;new time delay equal to time delay - 1      
    CMP R8, #0;  when time delay equal to zero; add 1 to register 5 and turn off the LED, if not leave it on    
    ADD R5, #1 
    BNE LED4
    BEQ turn_off_G
		
delay3
    SUB R8, R8, #1    
    CMP R8, #0
    ADD R5, #1
    BNE  LED2
    BEQ turn_off_B
	
delay2
    SUB R8, R8, #1    
    CMP R8, #0
    ADD R5, #1
    BNE  LED2
    BEQ turn_off_BL
	
delay1
    SUB R8, R8, #1   
    CMP R8, #0
    ADD R5, #1
    BNE  LED1
    BEQ turn_off_R ;this goes back to the initial turn off so as to create a looping effect when waiting for the user
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;END::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;SUBROUTINE FOR TURNING OF THE LEDs;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;this subroutine turns off each LED after the time delay which is stored in register 8 is equal to zero
turn_off_G ; turn off LED4 when 
	LDR R3,= GPIOA_ODR   ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
	LDR R4, [R3]
	ORR R4, #0xFFFFFFFF   ; this is funtion is to turn off any LED that is on, LED4
	STR R4, [R3]
	LDR R8, = time_delay
	B LED3


turn_off_B
	LDR R3,= GPIOA_ODR   ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
	LDR R4, [R3]
	ORR R4, #0xFFFFFFFF   ; this is funtion is to turn off any LED that is on, LED3
	STR R4, [R3]
	LDR R8, = time_delay
	B LED2
	
turn_off_BL
	LDR R3,= GPIOA_ODR   ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
	LDR R4, [R3]
	ORR R4, #0xFFFFFFFF   ; this is funtion is to turn off any LED that is on, LED2
	STR R4, [R3]
	LDR R8, = time_delay
	B LED1
	


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;END OF SUBROUTINE;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;	
	
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;; button subroutines;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;this subroutine is for the buttons associated with each LED greenbutton = LED4, blue button = LED3, black button= LED2, red button LED1
green_button
	LDR R6, = GPIOA_IDR
	LDR R7, [R6]
	LSR R7, #5
	AND R7, #1
	CMP R7, #0
	BEQ button_turn_off
    BX LR
	
blue_button
	LDR R6, = GPIOC_IDR
	LDR R7, [R6]
	LSR R7, #12
	AND R7, #1
	CMP R7, #0
	BEQ button_turn_off
    BX LR
	
black_button
	LDR R6, = GPIOB_IDR
	LDR R7, [R6]
	LSR R7, #9
	AND R7, #1
	CMP R7, #0
	BEQ button_turn_off
    BX LR

red_button
	LDR R6, = GPIOB_IDR
	LDR R7, [R6]
	LSR R7, #8
	AND R7, #1
	CMP R7, #0
	BEQ button_turn_off
    BX LR

;when the button for that LED is pushed the LEd should go off, is the purpose of this subroutine if not LED would remain on
button_turn_off
	LDR R3, = GPIOA_ODR            ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
    LDR R4, [R3]
    ORR R4, #0xFFFFFFFF       
    STR R4, [R3]
    B delay_game
	

delay_game
    SUB R8, R8, #1    
    CMP R8, #0; 
    ADD R5, #1
    BNE  button_turn_off
    BEQ seed


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;END OF BUTTON SUBROUTINE;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;random number generation;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;This generates the seed used for random number generation	
seed PROC
    mov R5, R12      ; this is where the seed is loaded into 
    ldr R9,= 0
	
random_number  ;This subroutine is used to generate random numbers 
    LDR R11,= 1664525           ; A
    LDR R10,= 1013904223        ; C
    MLA R12, R10, R12, R11     ;This uses the multiply and accumulate to generate the randome numbers (A * NEW SEED) + C
    LDR R1,= mole_delay      ; 
    LSR r12, #30          ; GET THE LAST TWO BITS     
    LDR R8, = game_delay  ;this is time before each random LED is lit 
    B gameplay

;when the last two bits of the random number generated which is in register 12 is gotten, if it is 00, LED4 comes on 
gameplay
    CMP R12, #0
    BEQ green_on
    
    CMP R12, #1   ; if it is 01 LED3 comes on 
    BEQ blue_on
    
    CMP R12, #2 ; if it is 10 LED2 comes on 
    BEQ black_on
    
    CMP R12, #3  ; if it is 11 LED1 comes on
    BEQ red_on    
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;END OF random number generation;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;;;;;Depending on the last two bits gotten from the randome number generated an LED comes on
green_on           ;turns on the LED that works for the green switch if 00
	 LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
	 LDR R4, [R3]
	 AND R4, #0xFFFFEFFF         ;FE= 1111 1110, turns on the LED4 PA12, LED is active low ;the 0 bit value signifies the LED being turned on
	 STR R4, [R3]
	 BL green_button1
	 B delay_green
    
blue_on        ;turns on the LED that works for the blue switch if 01
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFF7FF        ;F7 = 1111 0111, turns on the LED3 PA11
    STR R4, [R3]
    BL blue_button1
    B delay_blue
	
black_on        ;turns on the LED that works for the black switch if 10
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFFBFF        ;FB = 1111 1011,turns on the LED2
    STR R4, [R3]
    BL black_button1
    B delay_black
	
red_on            ;;turns on the LED that works for the red switch if 11
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFFEFF        ;FD = 1111 1101, LED is active low ;the 0 bit value signifies the LED being turned on
    STR R4, [R3]
    BL red_button1
    B delay_red	
	
;;;;;;; this tells the LED to go off after the button is pressed or a certain amount of time is the number is not pressed	
game_off      ;without this subroutine, all the lights would remain on at all times during the game 
    LDR R3, = GPIOA_ODR            ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
    LDR R4, [R3]
    ORR R4, #0xFFFFFFFF            ;0001 1110, TURNING OFF THE LEDS
    STR R4, [R3]
    SUB R1, R1, #1    
    CMP R1, #0
    BNE game_off
    BEQ nextround	

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;delay subroutine during the game;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; these are delay routines for the the game is in session 
delay_green
    SUB R8, R8, #1    
    CMP R8, #0
    ADD R5, #1
    BNE  green_on
    BEQ game_off
    
delay_blue
    SUB R8, R8, #1    
    CMP R8, #0
    ADD R5, #1
    BNE  blue_on
    BEQ game_off    

delay_black
    
    SUB R8, R8, #1    
    CMP R8, #0
    ADD R5, #1
    BNE  black_on
    BEQ game_off

delay_red
    SUB R8, R8, #1    
    CMP R8, #0
    ADD R5, #1
    BNE red_on
    BEQ game_off	

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;end of delay subroutine;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;game button subroutine;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;this subroutine is used to check if the button associated with each light has been pressed
green_button1
	LDR R6, = GPIOA_IDR
	LDR R7, [R6]
	LSR R7, #5
	AND R7, #1
	CMP R7, #0
	BEQ game_off
    BX LR
	
blue_button1
    LDR R6, = GPIOC_IDR        ;the address for the switch is 0x40010800 plus 8 for the input register
        ;this checks the input of the button
    LDR R7, [R6]
    LSR R7, #12
    AND R7, #1
    CMP R7, #0
    BEQ game_off
    BX LR
	
black_button1
    LDR R6, = GPIOB_IDR        ;the address for the switch is 0x40010800 plus 8 for the input register
        ;this checks the input of the button
    LDR R7, [R6]
    LSR R7, #9
    AND R7, #1
    CMP R7, #0
    BEQ game_off
    BX LR
	
red_button1
    LDR R6, = GPIOB_IDR        ;the address for the switch is 0x40010800 plus 8 for the input register
        ;this checks the input of the button
    LDR R7, [R6]
    LSR R7, #8
    AND R7, #1
    CMP R7, #0
    BEQ game_off
    BX LR
	
;;This subroutines is used to proceed to the next level when a level is completed
nextround
    ADD R9, R9, #1 ;this is used to track how many rounds have been played
    CMP R9, #10  ;check if ten rounds have been played
    mov R12, R5    
    BNE random_number ;generate random sequence is not equal to 10
    BEQ finished   ; end game if 10 levels have been played 

;; This turns off the LED when the game is over
final_turn_off
    LDR R3, = GPIOA_ODR            ;GPIOA_ODR    EQU        0x4001080C    ; (0x0C) Port Output Data Register
    LDR R4, [R3]
    ORR R4, #0xFFFFFFFF            ;0x00001e00 = 1e = ...0001 1110.... to make the LEDs disable
    STR R4, [R3]
    SUB R8, R8, #1    
    CMP R8, #0
    BEQ finished
    BNE final_turn_off


;this is the let the user know that they are done, All LED comes on 
finished
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFFDFF        ;FD = 1111 1101, LED is active low ;the 0 bit value signifies the LED being turned on
    STR R4, [R3]
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFFBFF        ;FB = 1111 1011
    STR R4, [R3]
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFF7FF        ;FB = 1111 1011
    STR R4, [R3]
    LDR R3, = GPIOA_ODR        ;GPIOC_ODR    EQU        0x4001100C
    LDR R4, [R3]
    AND R4, #0xFFFFEFFF        ;FB = 1111 1011
    STR R4, [R3]
    B success_delay
 
;this subroutine implements the wait time after a level 
success_delay
    SUB R8, R8, #1    ; change R4 to R2
    CMP R8, #0; change R4 to R2
    BNE  finished
    BL Reset_Handler


    ENDP
    ALIGN
		
;This routine will enable the clock for the Ports that needed    
GPIO_ClockInit PROC

    ; ENEL 384 Pushbuttons: SW2(Red): PB8, SW3(Black): PB9, SW4(Blue): PC12, SW5(Green): PA5
    ; ENEL 384 board LEDs: D1 - PA9, D2 - PA10, D3 - PA11, D4 - PA12
    LDR R0, = RCC_APB2ENR        ;enable register
    LDR R1, [R0]
    ORR R1, #0x1C                ;we OR this to turn on bits 1, 2 and 4, which enables PORT A, PORT B and PORT C, making them '1's
    STR R1, [R0]                ;store it back into the address of the APB

    BX LR
    ENDP
    ALIGN
;This routine enables the GPIO for the LED's.  By default the I/O lines are input so we only need to configure for ouptut. 
GPIO_init  PROC
    
    ; ENEL 384 board LEDs: D1 - PA9, D2 - PA10, D3 - PA11, D4 - PA12
    ; GPIOA_CRH    EQU        0x40010804    ; (0x04) Port Configuration Register for Px15 -> Px8
    ; changing the LED settings
    LDR R0, = GPIOA_CRH            ;loads the address of porta
    LDR R1, =0x44433334            ;gets address
    STR R1, [R0]
    
    BX LR
    ENDP
        

	ALIGN
    END
        
