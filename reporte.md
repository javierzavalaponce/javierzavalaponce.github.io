---
title: "Reporte actividades"
author: "Javier Zavala"
date: "2026-03-17"
---
bitacora:

# Actualizacion: 17 Mar 2026


## Emulador de sistemas dinamicos. Prototipo 0

Reporte de implementacion de sistemas dinamicos.
Primer prototipo.

## Descripcion del sistema

Sistema basado en microcontrolador
Entrada: señal de Audio tomada de PC

**Acondicionamiento de señal**


```bash 
              +5V
               │
             [R1 4.5k]
               │
               ●─────────> Vbias (~2.5V)
               │           Conectado a pin A0
Entrada ──||───┘           entrada a uControlador
audio     10uF
               │
             [R2 4.5k]
               │
              GND
```

* El capacitor de 10 µF bloquea el componente DC de la señal de audio.
* El divisor R1–R2 (4.5k + 4.5k) genera Voltaje Vbias =  2.5VDC.
* El audio queda *montado* sobre nivel Vbias.
* Desde el punto de vista de la señal de audio, el audio ve un pasa altas.
    * Frecuencia de corte vista por la señal. Fc=1/(2πRC)
    * Con R = 2.25kΩ , C = 100uF . Fc ~  7Hz
    * Por encima de ~7 Hz → la señal pasa casi sin atenuación.


**Diagrama a bloques**

```bash
      audio ───────┐
                   │
                ┌──┴───────┐
                │ EMULADOR │
                └──┬───────┘
                   │
                serial ────> logs via serial 115200 bps 8n1
```
* Frecuencia de **muestreo** fijada por Interrupion a **1KHhz**

**Señal de audio Chirp en Julia**

Este codigo genera una señal de audio para ejercitar
el emulador, se trata de una chirp escalonada (con silencios) para barrer frecuencia y demostrar la funcionalidad de un RC emulado en uControlador


```python
using WAV
using Plots

#Generar el chirp con silencios entre cada frecuencia
gr()
fs = 44100
frecuencias = [20, 30, 40, 50, 60, 70, 80, 90, 100]
# duración del silencio (20 ms)
silencio = zeros(Int(fs * 0.02))
chirp_signal = vcat([
    vcat(
        sin.(2π * fn .* (0:1/fs:(4/fn - 1/fs))),
        silencio
    )
    for fn in frecuencias
]...)

# vector de tiempo global
t = (0:length(chirp_signal)-1) ./ fs

p = plot(t, chirp_signal,
     xlabel="Tiempo (s)",
     ylabel="Amplitud",
     title="Barrido con silencios",
     label="señal")

savefig(p, "chirp.png")

# guardar archivo WAV
wavwrite(chirp_signal, "chirp.wav", Fs=fs)
println(length(chirp_signal) / fs, " segundos")
```


**Señal de audio Chirp en Julia**

```python
using WAV
using Plots

# Leer archivo
x, fs = wavread("chirp.wav")

# Parámetros RC
fc = 40.0
RC = 1 / (2π * fc)
T = 1 / fs
alpha = T / (RC + T)

# Filtro RC discreto
y = similar(x)
y[1] = x[1]


for n in 2:length(x)
    y[n] = y[n-1] + alpha * (x[n] - y[n-1])
end

# Graficar
t = (0:length(x)-1) ./ fs
plot(t, x, label="Entrada")
plot!(t, y, label="Salida (RC)")

# Guardar WAV
wavwrite(y, "filtered.wav", Fs=fs)
```
**Este es el filtro RC en julia:**
**y[n] = y[n-1] + alpha * (x[n] - y[n-1])**

**Codigo C de RC**

Solo se muestra la ISR ejecutada cada 1 ms

```c
ISR(TIMER1_COMPA_vect)
{
  static unsigned char signal = 0;
  // Filtro RC
  static float y = 0.0;
  const float alpha = 0.2;

  // Alternar pin para medir con analizador lógico
  signal = !signal;
  digitalWrite(LED_BUILTIN, signal ? HIGH : LOW);

  // Leer ADC y aplicar filtro RC: y[n] = alpha * x[n] + (1 - alpha) * y[n-1]
  digitalWrite(12,HIGH);
  // Leer ADC
  x = analogRead(pinADC);
  //centrar en 0 (Vbias de 2.5V)
  x = x - 512;
  // Aplicar filtro RC
  y = y + alpha * (x - y);
  sampleToSend = (int)(y + 512); // Recentrar para enviar valor entre 0-1023
  readyToSend = true; // Señalamos que hay dato listo
  digitalWrite(12,LOW);
}
```

## Imagenes

