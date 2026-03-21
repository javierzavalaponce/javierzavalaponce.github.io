---
title: "Reporte actividades"
author: "Javier Zavala"
date: "2026-03-17"
---
bitacora:

# Actualizacion: 17 Mar 2026


## Emulador de sistemas dinamicos. Prototipo 0

## Descripcion del sistema

* Entrada: señal de Audio tomada de PC
* Salida de datos por serial @ 115.2Kbps, 8n1.
* Periodo de muestreo Ts = 1 ms
    * *Proceso en tiempo real a 1 KHz*
* Sistema basado en microcontrolador. 


**Acondicionamiento de señal**

Acoplamiento señal analogica de audio 
*(Tratamiento antes de entrar a uControlador)*

```bash 
              +5V
               │
             [R1 4.5k]
               │
               |
               │    hacia el ADC
Entrada ──||───●──> Vbias (~2.5V) 
audio     10uF |    Entrada a uControlador
               │
             [R2 4.5k]
               │
              GND
```

* El capacitor de 10 µF bloquea el componente DC de la señal de audio.
* El divisor R1||R2 *(4.5k || 4.5k)* genera Voltaje **Vbias =  2.5 VDC**
* El audio queda *montado* sobre nivel Vbias.
* Desde el punto de vista de la señal de audio, el audio ve un pasa altas.
    * Frecuencia de corte vista por la señal. Fc=1/(2πRC)
    * Con R = 2.25kΩ , C = 100uF . Fc ~  7Hz (señales arriba de 7 Hz
      pasan sin atenuación)

**Diagrama a bloques**

```bash
      audio ───────┐
                   │
                ┌──┴───────┐
                │ EMULADOR │
                └──┬───────┘
                   │
                serial ────> logs via serial 
```
* Frecuencia de **muestreo** fijada por Interrupion de timer
 a **1KHhz**

**Generacion de archivo de audio chirp.wav en Julia: chirp.jl**

Chirp escalonada (con silencios) para barrer frecuencia con el fin de demostrar la funcionalidad de un RC emulado en uControlador. Audible desde cualquier reproductor.

```python
using WAV

#Generar el chirp con silencios entre cada frecuencia
fs = 44100

#Vector de frecuencias en  la señal chirp:
frecuencias = [20, 30, 40, 50, 60, 70, 80, 90, 100]

#Duración del silencio (20 ms) entre freq
silencio = zeros(Int(fs * 0.02))

#Genera chirp
chirp_signal = vcat([
    vcat(
        sin.(2π * fn .* (0:1/fs:(4/fn - 1/fs))),
        silencio
    )
    for fn in frecuencias
]...)


# guardar archivo WAV
wavwrite(chirp_signal, "chirp.wav", Fs=fs)
```
---


## Filtro RC en lenguaje julia rc.jl

Un RC es un filtro pasa bajas de primer orden
```c
                 R=1000;     C=.100uF
x(entrada)-> ---/\/\/\/\---| |-----GND
          -> i=Cy' ->     ^
                          |
                          y(Vout)
```                          
Si:
y = voltage Vout de capacitor


La corriende de la rama es 


i = Cy’


x = voltaje | audio de entrada , *Por Kirchhoff:*

```c
x = RCy’ + y
```

Reordenando: 
   RCy'=x-y o bien:

   ```c
   y'=(x-y)/RC          //(ecuacion 1)
   ```

   Por otro lado, la derivada es:

   ```c
   y'=(y[n-1]-y[n])/dt //(ecuacion 2)
```
   P ej. Para una frecuencia de muestreo fs de 1Khz
   *(1000 muestras / sec)*
   
   ```c
   dt=1/fs  //( 1 milisegundo)
   ```
   igualando ecuaciones 1 y 2:

   ```c
   y' = (y[n] +  x[n])   / RC       
   y' = (y[n] -  y[n-1]) / dt 
   ```
   Haciendo cambio de variable **alpha** con:
   
   ```c
   alpha = dt/(RC+dt)
   ```

   **alpha**

   * Versión discreta de la frecuencia de corte
   * Encapsula frecuencia de corte y muestreo

```python
using WAV

# Leer archivo
x, fs = wavread("chirp.wav")

# Vin ── R ──┬── Vout
#            |
#            C
#            |
#           GND
fc = 40.0
RC = 1 / (2π * fc)
T = 1 / fs
alpha = T / (RC + T)

# Filtro RC discreto
y = similar(x)
y[1] = x[1]


for n in 2:length(x)
    #rc();
    y[n] = y[n-1] + alpha * (x[n] - y[n-1])
    #traducir a lenguaje C...
end

# Guardar WAV
wavwrite(y, "chirp_despues_de_filtro_rc.wav", Fs=fs)
```

---

## Codigo C de RC (version tiempo real)

El siguiente codigo
muestra la (interrupcion) / ISR .
*Rutina de tiempo real*

Ejecutada deterministicamente 
(hard real time)
cada 1 ms:

**Prueba de concepto**

```c
#define V_BIAS (512) //ADC de 10bits
isr_1ms()
{
  // Filtro RC
  static float y = 0.0;
  static float alpha = 0.2;

  // Alternar pin para medir con analizador lógico
  // Tick de sistema de tiempo real
  ToggleLedISR_D0();

  //Mide tiempo de ejecucion:
  Set_D0_High();
  {
     // Leer ADC
     x = analogRead(pinADC);
   
     // Centrar en 0 (Vbias de 2.5V)
     x = x - V_BIAS;
   
     // Aplicar filtro RC
     y = y + alpha * (x - y);
   
     sampleToSend = (int)(y + V_BIAS); // Recentrar para enviar    valor entre 0-1023
     readyToSend = true; //  dato listo
  }
  Set_D0_Low();
}

```
---


## Imagenes

