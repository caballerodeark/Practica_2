# Pràctica 2: INTERRUPCIONES (Guillem Pérez, Vadym Lakymchuk)

## Practica A: Interrupcion por GPIO

Esquema de montaje:

![Esquema_Practica2a.png](https://file+.vscode-resource.vscode-cdn.net/Users/guillemperez/Desktop/Trabajo%20y%20apuntes%20uni/Q4/Procesadors%20Digitals/Practicas/P2/Esquema_Practica2a.png?nonce%3D1686063798108)

Script:

```cpp
#include <Arduino.h>

struct Button
{
    const uint8_t PIN;
    uint32_t numberKeyPresses;
    bool pressed;
};

Button button1 = {18, 0, false};

void IRAM_ATTR isr() 
{
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}

void setup() 
{
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() 
{
  if (button1.pressed) 
  {
    Serial.printf("Button 1 has been pressed %u times\n",button1.numberKeyPresses);
    button1.pressed = false;
  }
  //Detach Interrupt after 1 Minute
  static uint32_t lastMillis = 0;
  if (millis() - lastMillis > 60000) 
  {
    lastMillis = millis();
    detachInterrupt(button1.PIN);
    Serial.println("Interrupt Detached!");
  }
}
```

Funcionamiento: La placa detecta cuando el interruptor (button1) es presionado, en ese caso en el puerto serial escribe "Button 1 has been pressed (cantidad de veces presionado) times". En el caso de que transcurra 1 minuto, el pin del interruptor se interrumpe y deja de hacer la cuenta en este consecuentemente.

## Practica B: Interrupcion por Timer

Script:

```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;
 
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;
 
void IRAM_ATTR onTimer() 
{
  portENTER_CRITICAL_ISR(&timerMux);
  interruptCounter++;
  portEXIT_CRITICAL_ISR(&timerMux);
}
 
void setup() 
{
  Serial.begin(115200);
 
  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, 1000000, true);
  timerAlarmEnable(timer);
}
 
void loop() 
{
  if (interruptCounter > 0) 
  {
    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);
 
    totalInterruptCounter++;
 
    Serial.print("An interrupt as occurred. Total number: ");
    Serial.println(totalInterruptCounter);
  }
}
```

Funcionamiento: Este codigo genera un temporizador y realiza la cuenta de el numero de interrupciones generadas durante este tiempo, a traves de la declaracion de una alarma que salta cada vez que se realiza una interrupción sumandole a la cuenta de esta ademas.
