# whack-a-mole
what is the game: This game programmed is called Whack-A-Mole or in the case of the LEDs and buttons, Whack-A-LED with each associated buttons. 

How to play: There are 10 levels, as you increase in level the faster each LEDs illuminate. How the game is played is that when a mole comes up you have to hit it with something, for each mole you hit you get a point. 
But, in the case of the ENEL384 board soldered with LED and buttons and the STM32F10xx borad placed on top we are focusing on 
programming the LEd and the switches on the board to implement this game. I have programmed the LED and switches in such a way that each 
switch controls an LED, switch2 controls LED1, switch3 controls LED2, switch4 controls LED3, swicth5 controls LED4. I have also programmed 
the lights to randomly illuminate using the random number generation method given to us in class, when a light illuminates the switch which is associated with it ast stated
above should be pressed. This then sends a feedbacxk saying the LED has been hit (whacked). If when an LED illuminates and the switch associated with it is not pressed the
game should end as well display how many LED has been hit(whacked) in binary format using the 4 LEDs. IF a player is able to hit all LEDs at different rounds, the score woul 
be displayed when all rounds are completed.

I was able to alter the sequence in which each LEDs appear when waiting for the player to press a switch indicating they are ready. I was not able to program a register to record a score 
and display it at the end of the game using the 4 LEDs. I did not implement any feature beyond the basic requirements. My biggest problem was trying to get the LEDs to display the score, it was messing with my code so i took it out.
a possible future expansion would be manipulate the LEDs to display the score.


The fuction controlling the prelimwait is the time_delay function, if the user wants to reduce or increase wait time all you have tyo do is reduce of increase the number of the time_delay which is stored in register 8.
The react time is the mole_delay, the user can adjust this parameter by increasing or reducing the value of the mole_delay.
The number of cycles can be adjusted by changing the value stored in register 9. This fuction is implemented in nextround subroutine.
The values of winningSignalTime and LoosingSignalTime cannot be adjusted because this fuction, i was not able to implement.
       

