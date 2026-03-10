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

## En c sobre arduino:

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
## Respuesta en Frecuencia (repaso)

El siguiente script python muestra la
respuesta en frecuencia, es decir
que en vez de aplicar un escalon de 5Volts,
se aplica como entrada una funcion seno
de 5 Volts de amplitud y ademas se hace un barrido 
en frecuencia a fin de apreciar la respuesta en
frecuencia del RC.
se aplica 1Hz, 2Hz, 3Hz...10Hz
tambien se calcula la ganancia y se compara con
el valor teorico, todo con fines de *repaso y estudio
solamente.*


```perl

import math
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

RC = 0.1
Ts = 0.002
A  = 5.0  # Amplitud

# Vector de frecuencias
f = np.linspace(1, 10, 10)

# Tiempo de simulación (500 puntos = 1 segundo)
t = np.arange(0, 1.0, Ts)

# Lista para almacenar todas las simulaciones y[]
Y_matrix = []
gain = []

for i, freq in enumerate(f):
    x = A * np.sin(2 * np.pi * freq * t)  # Entrada seno (vector)
    y = np.zeros_like(t)  # Estado inicial
    for n in range(1, len(t)):
            y[n] = y[n-1] + (Ts/RC) * (x[n] - y[n-1])  # Ecuación discreta RC
    
    Y_matrix.append(y)  # Agregar fila completa
    amp_out = (np.max(y) - np.min(y))/2
    gain.append(amp_out/A)

# Convertir a matriz 10x500 y luego DataFrame
Y_array = np.array(Y_matrix)  # Shape: (10, 500)
df = pd.DataFrame(Y_array, 
                  index=[f"Freq_{int(fi)}Hz" for fi in f],
                  columns=[f"t_{i}" for i in range(500)])

print("Shape del DataFrame:", df.shape)
print(df.head())

# GRAFICAR: 10 curvas en mismo plot
plt.figure(figsize=(12, 8))
for freq_label in df.index:
    plt.plot(t, df.loc[freq_label], label=freq_label, linewidth=2)

plt.xlabel('Tiempo [s]')
plt.ylabel('Voltaje salida [V]')
plt.title('Respuesta en Frecuencia del Circuito RC\n(10 frecuencias 1-10 Hz)')
plt.grid(True, alpha=0.3)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.savefig("01.png", dpi=300)


f_theory = np.linspace(1,10,200)
H = 1/np.sqrt(1 + (2*np.pi*f_theory*RC)**2)
plt.figure()
plt.plot(f, gain, 'o', label="Simulación")
plt.plot(f_theory, H, label="Teoría")
plt.legend()
plt.xlabel("Frecuencia [Hz]")
plt.ylabel("Ganancia")
plt.title("Respuesta en frecuencia del filtro RC")
plt.grid(True)
plt.tight_layout()
plt.savefig("02.png", dpi=300)

```




## Imagenes

