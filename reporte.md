---
title: "Reporte actividades"
author: "Javier Zavala"
date: "2026-03-09"
---
bitacora:

# Actualizacion: 9 Mar 2026


## Simulacion de RC 

```perl
                 R=1K     C=100uF
(entrada)x->+---/\/\/\/\---| |-----GND
                          ^
                          |
                          y(salida)
```

Si:

y = voltage de capacitor

i = Cy'

x = entrada

Entonces: x = RCy' + y

---

## En python:

```perl

## simulacion de RC en python

y = 0.0; #salida
x = 5.0; #entrada , aplica escalon de 5 Volts

RC = 0.1;   # R 1K, C 100u
Ts = 0.002; # Simula una muestra cada 2 ms 

for i in range(500):
    y = y + (Ts/RC)*(x - y);
    t = i*Ts #tiempo
    print(f"{t:.4f},{y:.4f}")

```
___

## En c sore arduino:

```c
float y = 0.0;
float x = 5.0; // Aplica escalon de 5 Volts

float RC = 0.1;
float Ts = 0.002;

unsigned long last = 0;

void setup()
{
    Serial.begin(115200);
}

void loop()
{   //toma una muestra cada 2 ms 
    if (micros() - last >= 2000)
    {       
        last = micros();
        y = y + (Ts/RC)*(x - y); //<<--AQUI!!!!
        Serial.print(x);
        Serial.print(",");
        Serial.println(y);
    }
}
```
___


```perl
#asi para Graficar
import serial
import matplotlib.pyplot as plt

ser = serial.Serial('/dev/ttyUSB0',115200)

x=[]
y=[]

for i in range(500):
    line=ser.readline().decode().strip()
    y.append(float(line))

plt.plot(y,color="green")
plt.title("Simulación Circuito RC en arduino")
plt.grid()
plt.tight_layout()
plt.savefig("file_arduino.png", dpi=300)

```

## Imagenes

