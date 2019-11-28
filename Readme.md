# Interrupción UART - Simulación I/O de UART en MPLAB X

## UART 1

### Configurar UART 1

Para utilizar la interrupción del UART 1, primero debe configurarse correctamente, la función siguiente ejemplifica una configuración estándar para enviar y recibir a datos.

```assembly
MOV    #baudrate,W0
MOV    W0,U1BRG ; Set Baudrate
BSET IPC2,#U1TXIP2 
BCLR IPC2,#U1TXIP1 
BCLR IPC2,#U1TXIP0
BSET IPC2,#U1RXIP2 
BCLR IPC2,#U1RXIP1 
BCLR IPC2,#U1RXIP0 
CLR    U1STA
MOV    #0x8800,W0
MOV    W0,U1MODE			; Enable UART for 8-bit data, ; no parity, 1 STOP bit,
BSET U1STA,#UTXEN			; Enable transmit
BSET   IEC0,#U1TXIE		; Enable transmit interrupts 
BSET   IEC0,#U1RXIE		; Enable receive interrupts
```

> Esta información se encuentra en el **DSPIC30F FAMILY REFERENCE MANUAL**, en la seccion 19







###  La rutina **__U1RXInterrupt**

Los nombres de las rutinas de interrupción ya estan definidos en su mayoría, en este caso la rutina **__U1RXInterrupt** se ejecutara cuanto la interrupción RX ocurra, en general estas rutinas siguen la siguiente estructura básica.

```assembly
__U1RXInterrupt:
    
    ; YOUR CODE
    
    BCLR    IFS0,	#U1RXIE
    RETFIE
```

###  La rutina **__U1TXInterrupt**

El envio de información por medio del UART no tiene un nombre de rutina asignado, en general, cada que U1TXREG es modifcado, esta informacion se envía, sin embargo, para enviar una cantidad considerable de datos (como una cadena de texto) lo mas recomendable es implementar una rutina para este fin.

```assembly
__U1TXInterrupt:
    tblrdl.b        [W9++],W10       ; Read a next string byte to w0 and advance a string pointer w1     
      cp0.b   W10                  ; Check and exit if the byte equals to zero     
      bra     z,_rs232_tx_ret         ;            ;
_rs232_tx_byte:
       btsc    U1STA,#UTXBF            ; Check if UART1 TX buffer is empty
       bra     _rs232_tx_byte          ; Keep checking if not yet
       mov     W10,U1TXREG
       MOV	0x00, W10; Send a byte via UART1
       bra     __U1TXInterrupt        ; Loop until all non-zero bytes are transmitted  
_rs232_tx_ret:
			 RETURN
```

> Esta función extrae el mensaje a enviar de la memoria del programa, cuya dirección de inicio fue almacenada previamente en W9

## Simular I/O de UART en MPLAB X

Para simular la entrada y salida de datos a traves UART, es necesario realizar varias tareas.

Para activar la simulación de la transmisión de datos desde el PIC:

1. Clic en **Production**  > **Set project configuration** > **Customize**
2. Se abre la ventana **Project properties**, dar click en el campo **Simulator**
3. En **Option categories** seleccionar la opción **UART1 I/O Options**
4. Activar la casilla **Enable UART I/O Options**
5. Seleccionar la forma de salida (En consola o en un archivo de texto)

Para activar la simulación de la recepción de datos en el PIC:

1. Clic en **Window** > **Simulator** > **Stimulus**

2. Se abre la ventana **Stimulus**, dar click en la pestaña **Register Injection **

3. Seleccionar el registro que nos interese controlar, en este caso U1RXREG

4. En el campo **Trigger**, seleccionar **Message**

5. Seleccionar el archivo que contiene las entradas que se quieren simular, el ejemplo siguiente muestra el contenido de uno de estos archivos.

   ```javascript
   wait 20000 ms
   "1"
   wait 10000 ms
   "2"
   ```

------

6. En el campo **Data Filename** cargar el archivo
7. Iniciar la simulación, click en **Continue**
8. Mientras la simulación este activa, la consola o el archivo seleccionado en la configuración del proyecto se modificará con las respuestas del PIC (Mensajes enviados via **U1TXREG**)