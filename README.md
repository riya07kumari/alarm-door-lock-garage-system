# alarm-door-lock-garage-system
this is my microcontroller microprocessor project 
PROGRAM CODE- 
ORG 0000H 
 
PASSWORD_BUFFER EQU 30H    ; Memory location to store password buffer 
PASSWORD_ATTEMPT EQU 40H     ; Memory location to store the number of attempts 
MOTOR1_FORWARD      EQU     P2.4   ; Motor direction - forward (open)-IN2 
MOTOR1_BACKWARD     EQU     P2.3    ; Motor direction - backward (close)-IN1 
MOTOR1_ENABLE       EQU     P2.2    ; Motor enable-ENA 
MOTOR2_FORWARD      EQU     P2.5    ; Motor direction - forward (open)-IN4 
MOTOR2_BACKWARD     EQU     P2.6    ; Motor direction - backward (close)-IN3 
MOTOR2_ENABLE       EQU     P2.7    ; Motor enable-ENB 
IR1 EQU P3.5     ; IR SENSOR 1 
IR2 EQU P3.6     ; IR SENSOR 2 
SETB P3.7 
 
START: 
 MOV R0, #PASSWORD_BUFFER   ; To store the password buffer in R0 register 
 MOV R1, #PASSWORD_ATTEMPT   ; To store the number of password attempts in R1  
 MOV @R1, #3   ; Here, R1 stores the value 3 
SJMP COMMAND 
 
COMMAND:      ; Label for executing LCD commands one by one 
 MOV DPTR, #LCD_COMMAND 
 MOV R2, #5 
C1: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_COMNWRT 
 INC DPTR 
 DJNZ R2, C1 
 ACALL DELAY_1S 
 SJMP WELCOME 
  
WELCOME:     ; Label to display ‘WELCOME’ message on LCD display 
 MOV A, #84H   ; To position the cursor at 4th position, first line 
 ACALL LCD_COMNWRT 
 MOV DPTR, #LCD_WELCOME 
 MOV R2, #8    ; R2 stores the length of the data byte so as to print one letter at a time 
D1:  
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D1 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S 
 ACALL LCD_CLEAR 
 SJMP ENTER_PASSWORD 
 
ENTER_PASSWORD: ; Label to display ‘ENTER PASSWORD’ message 
 ACALL COMMAND2 
 MOV A, #80H     ; To position the cursor at first line, 0th position 
 ACALL LCD_COMNWRT 
 MOV DPTR, #LCD_ENTER_PASSWORD 
 MOV R2, #15   ; R2 stores the length of the data byte so as to print one letter at a time 
D2: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D2 
 ACALL DELAY_71MS  
 MOV A, #0C6H     ; To position the cursor at second line, 6th position 
 ACALL LCD_COMNWRT 
 SJMP LENGTH 
 
LENGTH: MOV R2, #4    ; R2 stores the value 4 which is the password length 
KEYPAD_START: MOV P0, #0FFH  
 
K1:     ; Checking whether all keys are released 
 MOV P3, #0  ;ROWS - PORT3 
 MOV A, P0  ;COLUMNS - PORT0 
 ANL A, #00001111B 
 CJNE A, #00001111B,K1 
 
K2:      ;Checking whether any key is pressed 
 ACALL DELAY_71MS 
 MOV A,P0  
 ANL A,#00001111B  
 CJNE A,#00001111B,OVER  
 SJMP K2  
 
OVER:     ;Checking whether the key is pressed and moves to check the row  
 ACALL DELAY_71MS 
 MOV A,P0  
 ANL A,#00001111B  
 CJNE A,#00001111B,OVER1 
 SJMP K2  
 
OVER1:  
 MOV P3,#11111110B  
 MOV A,P0  
 ANL A,#00001111B  
 CJNE A,#00001111B,ROW_0     ; If A holds the value 00001110, then the pointers moves 
to ROW_0 
 MOV P3,#11111101B  
 MOV A,P0  
 ANL A,#00001111B  
 CJNE A,#00001111B,ROW_1     ; If A holds the value 00001101, then the pointers moves 
to ROW_1 
 MOV P3,#11111011B  
 MOV A,P0  
 ANL A,#00001111B  
 CJNE A,#00001111B,ROW_2    ; If A holds the value 00001011, then the pointers moves 
to ROW_2 
 MOV P3,#11110111B 
 MOV A,P0 
 ANL A,#00001111B 
 CJNE A,#00001111B,ROW_3    ; If A holds the value 00000111, then the pointers moves 
to ROW_0 
 LJMP K2    ; If none of the keys are pressed, jump to K2 
 
ROW_0:  
 MOV DPTR, #KCODE0  
 SJMP FIND     
ROW_1:  
 MOV DPTR, #KCODE1  
 SJMP FIND  
ROW_2:  
 MOV DPTR, #KCODE2  
 SJMP FIND  
ROW_3:  
 MOV DPTR, #KCODE3  
 SJMP FIND 
 
FIND:    ; Label to find the corresponding column of the row containing the key pressed 
 RRC A     ; If RRC A results in a carry, increment the DPTR and repeat the process,  
 JNC MATCH     ; Else, if no carry is generated, jump to MATCH to display the key pressed 
 INC DPTR  
 SJMP FIND  
MATCH:      
 CLR A 
 MOVC A,@A+DPTR 
 ACALL LCD_DATAWRT 
 MOV @R0, A     ; To store the key value to password buffer 
 INC R0    ; Increment R0 for storing the next key value in subsequent memory location 
 DJNZ R2, K1    ; To enter the digits four times  
 MOV A, #0CH      ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S  
 ACALL LCD_CLEAR 
 ACALL DELAY_1S 
 SJMP PROCESS 
 
PROCESS: ; Label to display ‘VERIFYING’ message 
 ACALL COMMAND2 
 MOV A, #82H    ; To position the cursor at first line, 2nd  position 
 ACALL LCD_COMNWRT 
 MOV DPTR, #PROCESSING 
 MOV R2, #12    ; R2 stores the length of the data byte so as to print one letter at a time 
D6: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR  
 DJNZ R2, D6 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S 
 SJMP PASSWORD_LENGTH 
 
PASSWORD_LENGTH:    ; Label to initialize password verification 
 ACALL COMMAND2 
 MOV R7, #4 
 MOV R0, #PASSWORD_BUFFER 
 MOV DPTR, #PASSWORD 
VERIFY_PASSWORD:    ; Label to verify the password 
 MOV B, @R0    ; To load the value at R0 to B 
 CLR A 
 MOVC A, @A+DPTR   ; Accumulator stores the actual password stored in DPTR 
 CJNE A, B, INCORRECT_PASSWORD    ; If value stored in A and B are not equal, jump to 
INCORRECT_PASSWORD 
 INC R0    ; Else, increment R0 and DPTR for verifying next value 
 INC DPTR 
 DJNZ R7, VERIFY_PASSWORD    
 SJMP CORRECT_PASSWORD 
 
CORRECT_PASSWORD:    ; Label to display ‘Correct Password’ message 
 ACALL LCD_CLEAR 
 ACALL COMMAND2 
 MOV DPTR, #CORRECT_MESSAGE 
 MOV A, #84H    ; To position the cursor at first line, 4th  position 
 ACALL LCD_COMNWRT 
 MOV R2, #7     ; R2 stores the length of the data byte so as to print one letter at a time 
D3: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR  
 DJNZ R2, D3 
 MOV A, #0C3H    ; To position the cursor at second line, 3rd  position 
 ACALL LCD_COMNWRT 
 MOV R2, #9    ; R2 stores the length of the data byte so as to print one letter at a time 
D8: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D8 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S 
 ACALL LCD_CLEAR 
 LJMP DOOR_OPEN 
 
INCORRECT_PASSWORD:     ; Label to display ‘Incorrect Password’ message 
 DEC @R1   ; Decrements the number of attempts by 1 for every incorrect password 
 ACALL LCD_CLEAR 
 ACALL COMMAND2 
 MOV DPTR, #INCORRECT_MESSAGE 
 MOV A, #83H    ; To position the cursor at first line, 3rd  position 
 ACALL LCD_COMNWRT 
 MOV R2, #9    ; R2 stores the length of the data byte so as to print one letter at a time 
D4: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D4 
 MOV A, #0C3H    ; To position the cursor at second line, 3rd  position 
 ACALL LCD_COMNWRT 
 MOV R2, #9   ; R2 stores the length of the data byte so as to print one letter at a time 
D9: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D9 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S 
 ACALL LCD_CLEAR 
 CJNE @R1, #0, TRY_MESSAGE     ; If any attempt is left, the pointer jumps to 
TRY_MESSAGE 
 SJMP BUZZER_ALERT    ; Else, the pointer jumps to BUZZER_ALERT to trigger the buzzer 
 
TRY_MESSAGE: 
 ACALL COMMAND2 
 MOV A, #83H    ; To position the cursor at first line, 3rd  position 
 ACALL LCD_COMNWRT 
 MOV DPTR, #TRY_AGAIN 
 MOV R0, #10    ; R0 stores the length of the data byte so as to print one letter at a time 
D5: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR  
 DJNZ R0, D5 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S 
 ACALL LCD_CLEAR 
 SJMP ATTEMPTS_LEFT 
 
ATTEMPTS_LEFT:    ; Label to display the number of attempts left 
 ACALL COMMAND2 
 MOV A, #84H    ; To position the cursor at first line, 4th  position 
 ACALL LCD_COMNWRT 
 MOV R0, #8   ; R0 stores the length of the data byte so as to print one letter at a time 
 MOV DPTR, #ATTEMPT_LEFT 
D12: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R0, D12 
 MOV A, #0C5H    ; To position the cursor at second line, 5th position 
 ACALL LCD_COMNWRT 
 MOV R0, #5    ; R0 stores the length of the data byte so as to print one letter at a time 
D13: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R0, D13 
 SJMP DISPLAY_NUMBER 
  
DISPLAY_NUMBER: 
 MOV DPTR, #ATTEMPT_NUMBER 
 MOV A, #0CAH    ; To position the cursor at second line, 10th  position 
 ACALL LCD_COMNWRT 
 CLR A 
 MOV A, @R1 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_1S 
 ACALL LCD_CLEAR 
 LCALL CLEAR_BUFFER 
 LJMP ENTER_PASSWORD 
 
BUZZER_ALERT:    ; Label to display ‘Attempts over’ message and trigger the buzzer alert 
 ACALL LCD_CLEAR 
 ACALL COMMAND2 
 MOV DPTR, #ATTEMPT_OVER 
 MOV A, #84H    ; To position the cursor at first line, 4th position 
 ACALL LCD_COMNWRT 
 MOV R2, #8     ; R2 stores the length of the data byte so as to print one letter at a time 
D10: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D10 
 MOV A, #0C5H    ; To position the cursor at second line, 5th  position 
 ACALL LCD_COMNWRT 
 MOV R2, #5    ; R2 stores the length of the data byte so as to print one letter at a time 
D11: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR 
 DJNZ R2, D11 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 CLR P3.7    ; To turn on the buzzer for a particular time 
 ACALL DELAY_10S 
 SETB P3.7 
 LJMP CLOSE 
  
DOOR_OPEN:    ; Label to display ‘Door opens’ message 
 ACALL COMMAND2 
 MOV A, #83H    ; To position the cursor at first line, 3rd  position 
 ACALL LCD_COMNWRT 
 MOV DPTR, #OPEN_DOOR 
 MOV R0, #10     ; R0 stores the length of the data byte so as to print one letter at a time 
D7: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR  
 DJNZ R0, D7 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 ACALL DELAY_2S 
 SJMP MOTOR_OPEN 
 
MOTOR_OPEN:    ; Label to operate the motors and open the door 
 SETB MOTOR1_BACKWARD 
 SETB MOTOR2_BACKWARD 
 CLR MOTOR1_FORWARD 
 CLR MOTOR2_FORWARD 
 MOV R2, #1 
 LOOP1: 
 MOV R0, #100 
 OPEN_PWM_LOOP: 
  ACALL PWM_ON 
  ACALL PWM_OFF 
  DJNZ R0, OPEN_PWM_LOOP 
  ACALL DELAY_1S 
  DJNZ R2, LOOP1 
 CLR MOTOR1_ENABLE 
 CLR MOTOR2_ENABLE 
 ACALL LCD_CLEAR 
 MOTION1:     ; Once door is opened, the IR sensors would be ON and wait until motion is  
  JNB IR1, MOTION1    ; detected. Once both the sensors are triggered, the door 
closes automatically 
 MOTION2: 
  JNB IR2, MOTION2 
 PROCEED: 
  ACALL DELAY_5S 
 SJMP DOOR_CLOSE 
 
DOOR_CLOSE:    ; Label to display ‘Door closes’ message 
 ACALL COMMAND2 
 MOV A, #83H    ; To position the cursor at first line, 3rd position 
 ACALL LCD_COMNWRT 
 MOV DPTR, #CLOSE_DOOR 
 MOV R0, #11    ; R0 stores the length of the data byte so as to print one letter at a time 
D14: 
 CLR A 
 MOVC A, @A+DPTR 
 ACALL LCD_DATAWRT 
 INC DPTR  
 DJNZ R0, D14 
 MOV A, #0CH    ; Command to make LCD ON and cursor OFF 
 ACALL LCD_COMNWRT 
 SJMP MOTOR_CLOSE 
 
MOTOR_CLOSE:    ; Label to operate the motors and close the door 
 CLR MOTOR1_BACKWARD 
 CLR MOTOR2_BACKWARD 
 SETB MOTOR1_FORWARD 
 SETB MOTOR2_FORWARD 
 MOV R2, #10 
 LOOP2: 
 MOV R0, #10 
 CLOSE_PWM_LOOP: 
  ACALL PWM_ON 
  ACALL PWM_OFF 
  DJNZ R0, CLOSE_PWM_LOOP 
  ACALL DELAY_1S 
  DJNZ R2, LOOP2 
 CLR MOTOR1_ENABLE 
 CLR MOTOR2_ENABLE 
 ACALL LCD_CLEAR 
 MOV A, #08H      ; Command to make LCD off 
 ACALL LCD_COMNWRT 
 ACALL DELAY_5S 
 LJMP START 
 
CLOSE:     ; Label for closing the lock after door is closed 
 MOV A, #08H        ; Command to make LCD off 
 ACALL LCD_COMNWRT 
 ACALL DELAY_20S 
 LJMP START    ; To repeat the whole process again 
 
LCD_COMNWRT:    ; Label for executing the command for LCD 
 MOV P1, A  
 CLR P2.0      ; RS 
 SETB P2.1     ; EN 
 ACALL DELAY_71MS 
 CLR P2.1 
 RET 
 
LCD_DATAWRT:    ; Label for executing the display commands for LCD 
 MOV P1, A  
 SETB P2.0     ; RS 
 SETB P2.1     ; EN 
 ACALL DELAY_71MS 
 CLR P2.1 
 RET 
  
DELAY_71MS:     ; Delay subroutine for 71ms 
 MOV R3, #255 
 HERE1: MOV R4, #255 
 HERE2: DJNZ R4, HERE2 
    DJNZ R3, HERE1 
    RET 
 
DELAY_1S:      ; Delay subroutine for 1s 
 MOV R3, #14 
 HERE3: MOV R4, #255 
 HERE4: MOV R5, #255 
 HERE5: DJNZ R5, HERE5 
    DJNZ R4, HERE4 
    DJNZ R3, HERE3 
    RET 
 
DELAY_10S:       ; Delay subroutine for 10s 
 MOV R6, #10 
 HERE6:  
  ACALL DELAY_1S 
  DJNZ R6, HERE6 
 RET 
 
DELAY_20S:      ; Delay subroutine for 20s 
 MOV R6, #20 
 HERE7: 
  ACALL DELAY_1S 
  DJNZ R6, HERE7 
 RET 
 
DELAY_5S:     ; Delay subroutine for 5s 
 MOV R6, #5 
 HERE8:  
  ACALL DELAY_1S 
  DJNZ R6, HERE8 
 RET 
 
DELAY_2S:       ; Delay subroutine for 2s 
 MOV R6, #2 
 HERE9:  
  ACALL DELAY_1S 
  DJNZ R6, HERE9 
 RET 
 
LCD_CLEAR:         ; Command for clearing the LCD 
 MOV A, #01H 
 ACALL LCD_COMNWRT 
 RET 
 
CLEAR_BUFFER:       ; Label for clearing the buffer 
 MOV R0, #PASSWORD_BUFFER 
 RET 
 
PWM_ON:        ; Label for setting PWM such that motors are working with ON efficiency of 75% 
 SETB MOTOR1_ENABLE 
 SETB MOTOR2_ENABLE 
 MOV R3, #5 
ON_DELAY_LOOP: 
 MOV R4, #255 
PWM_1: DJNZ R4, PWM_1 
 DJNZ R3, ON_DELAY_LOOP 
 RET 
 
PWM_OFF:     ; Label for setting PWM such that motors are working with OFF efficiency of 25% 
 CLR MOTOR1_ENABLE 
 CLR MOTOR2_ENABLE 
 MOV R3, #3 
OFF_DELAY_LOOP: 
 MOV R4, #255 
PWM_2: DJNZ R4, PWM_2 
 DJNZ R3, OFF_DELAY_LOOP 
 RET 
 
COMMAND2:        ; LCD commands  
 MOV A, #0FH     ; Command to activate LCD and cursor and ON cursor blinking 
 ACALL LCD_COMNWRT 
 MOV A, #06H     ; Command to increment the cursor 
 ACALL LCD_COMNWRT 
 MOV A, #01H    ; Command to clear the LCD 
 ACALL LCD_COMNWRT 
 RET 
 
ORG 0300H        ; Data bytes are stored starting from 300H memory locations 
PROCESSING: DB 'V','E','R','I','F','Y','I','N','G','.','.','.' 
OPEN_DOOR: DB 'D','O','O','R',' ','O','P','E','N','S' 
CLOSE_DOOR: DB 'D','O','O','R',' ','C','L','O','S','E','S' 
ATTEMPT_OVER: DB 'A','T','T','E','M','P','T','S','O','V','E','R','!' 
ATTEMPT_LEFT: DB 'A','T','T','E','M','P','T','S','L','E','F','T',':' 
ATTEMPT_NUMBER: DB '0','1','2' 
PASSWORD: DB '1','2','3','4' 
INCORRECT_MESSAGE: DB 'I','N','C','O','R','R','E','C','T','P','A','S','S','W','O','R','D','!' 
CORRECT_MESSAGE: DB 'C','O','R','R','E','C','T','P','A','S','S','W','O','R','D','!' 
TRY_AGAIN: DB 'T','R','Y',' ','A','G','A','I','N','!' 
LCD_COMMAND: DB 38H, 0FH, 06H, 01H, 84H 
LCD_WELCOME: DB 'W','E','L','C','O','M','E','!' 
LCD_ENTER_PASSWORD: DB 'E','N','T','E','R',' ','P','A','S','S','W','O','R','D',':' 
KCODE3: DB '*','0','#','D' 
KCODE2: DB '7','8','9','C' 
KCODE1: DB '4','5','6','B'  
KCODE0: DB '1','2','3','A'  
END 
