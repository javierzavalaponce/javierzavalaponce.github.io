Ultima actualizacion: 2 Marzo 2026

## Dev Boards NXP
* Ambas FRDM-KL28Z (rojas) estan defectuosas, no permiten programacion. Estan bloqueadas.
    * El hardware se encuentra en un modo de seguridad parcial que impide 
     la configuración de cualquier periférico, lo que la inutiliza por completo 
     para fines de desarrollo y programación.
* FRDM - K64 - OK, toolchain instalado (GCC+cmake+Segger Jlink para debug)
    * Perifericos habilitados hasta ahora: Gpios y uart
    * Nota: el arbol de relojes es complicado sin los helpers del IDE oficial
    * **Demo** de avances programado para semana del 9-13 Marzo 2026
* Siguiente paso: integracion de FreeRTOS, evaluacion de Zephyr.

## Tarjeta DAC7564 
* Falta conexion de pin12, este pin alimenta la logica digital, imposible que funcione SPI
* Pregunta: : **Es la unica version de HW?**

## Bison y Flex
* En fase de aprendizaje




