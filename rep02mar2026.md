---
title: "Reporte actividades"
author: "Javier Zavala"
date: "2026-03-03"
---

# Actualizacion: 2 Marzo 2026

## Dev Boards NXP
* Ambas FRDM-KL28Z (rojas) estan defectuosas, no permiten programacion. Estan bloqueadas.
    * En el caso de la ultima que me paso, el diagnostico es el siguiente: 
      **El hardware se encuentra en un modo de seguridad parcial que impide 
        la configuración de cualquier periférico, lo que la inutiliza por completo 
        para fines de desarrollo y programación.**
* Por lo cual, he seguido trabajando con la FRDM - K64, en resumen:
    * Toolchain instalado (GCC+cmake+Segger Jlink para debug) sobre Debian 13
    * Perifericos habilitados hasta ahora: GPIOS y UART
    * Nota: el arbol de relojes es complicado sin los helpers del IDE oficial, (
      *aun me falta profundizar en este modulo de cfg de reloj*)
    * **Demo** de avances programado para semana del 9-13 Marzo 2026.
      Puedo presentar avances de lo que he logrado con esta board.
* Siguiente paso: integracion de FreeRTOS, evaluacion de Zephyr.

___
## Tarjeta DAC7564 
* **IMPORTANTE:** Falta conexion de pin12, este pin alimenta la logica digital, imposible que funcione SPI interno. Señalo que he estado utilizando analizador lógico
(Inclusive el documento de JL muestra el pin flotado)
* Pregunta: : **Es la unica version de HW?**

___
## Bison y Flex
* En fase de aprendizaje

___
## Emulación de sistemas dinámicos

En paralelo, voy avanzando, (repasando y reafirmando )
conceptos, y desenpolvando mis libros.

Antes de pasar a traductor a C, estoy usando 
python y ngspice, por ejemplo



```bash 
$ ngspice rc.cir
 ```

Este es el contenido de rc.cir

```perl
 * RC circuit simulation
* R = 1k Ohm
* C = 100uF

V1 in 0 PULSE(0 5 0 1u 1u 1 2)
R1 in out 1000
C1 out 0 100u

.tran 0.01 1
.control
run
plot v(out)
.endc

.end



```

___
## Imagenes

<!--
| Columna 1 | Columna 2 | Columna 3 |
| :--- | :---: | ---: |
| Izquierda | Centro | Derecha |
| Fila 2 | Datos | Datos |
-->


