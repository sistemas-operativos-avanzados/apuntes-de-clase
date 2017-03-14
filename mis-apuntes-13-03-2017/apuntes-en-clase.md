Apuntes del dia 13 de marzo 2017
================================

Se hizo el quiz #4 (revisar las preguntas en el celular)

Se hizo una revisión del proyecto 2 - Threadville

Para el examen un forro hecho a mano.

Resumenes para el 20 de marzo 

0311
0320 * lo expongo yo
0401
0402
0406
0608 *

## Monitores de Máquinas Virtuales (VMM)

Ejemplo: simular una arquitectura en otra, por ejemplo ARM en x86
Cómo seria ese programa?
El programa es un intéprete, se escribiría un ciclo de _FETCH_
Hay que fijarse en los bites del código de operación y hacer algo en C que hagan el mismo _shift_ a la izquierda
Probablemente en el corazón de este programa habria un 'switch/case' muy grande con todos los codigos de operacion

```
switch(codigo_operacion)
    SHL: ...
         ...
         break;
    ADD: ...
         ...
         break;
```

En ensamblador el ejercicio seria relativamente similar, en una parte está el bloque que hace el shift a la izquierda en ARM y se le llega a él por medio de _jumps_.

#### Si las dos arquitecturas son muy diferentes? nada en común?
la simulacion de cada instruccion seria muy grande y complicada

#### Que tal si no son tan diferentes?
El conjunto de instrucciones se hace más pequeño

Arquitecturas mas parecidas = simulación más fácil y corta

#### Y si hubiera que hacer un simulador de x86 en x86?
Seria la misma instruccion para simular la instruccion del otro lado.

#### Cómo hacer un simulador de una arquitectura en ella misma? se deja que corra sobre el mismo hardware. 
- Lo que ocurre tiene que ser aislado. 
- Tiene que ser perfecta, cualquier sofware que corra en el hardware real tiene que correr en la simulada

**El verdadero reto en VM es ejecutar una arquitectura en si misma**

La proteccion al final es un concepto de hardware.

El SO al final lo que hace es reestringuir la simulacion porque restringue al programa que se esta ejecutando

Esta idea esta a partir de los 70s
Tuvo mucho actividad al principio y luego paso de moda
Se considero algo muy académico

El concepto es muy elegante e interesante en SO

## Como escribir un VMM?

Hay hardware que es el verdadero
sobre ese hardware va a correr un software, el VMM
Este software va a crear varias máquina virtuales 
Requisitos:
- Los H' son indenticos al hardware que esta abajo el H
- Cualquier sofware que corre en el hardware real deberia de correr en las virtualizadas
- Un software que interesaria que corriera en H' es un SO. 
- Podrian ser diferentes SOs
- Cada uno esta totalmente aislado de las otras VMs
- **VMM** 
    - va a repartir el tiempo de las VMs, le va a dar un poco tiempo para correr
    - Se va a procurar de proteccion y no de administracion (Como en Exokernel), las va a aislar, no pueden ser afectadas por las otras
    - Tienen que correr en modo protegido? se puede resolver quitando el SO. El VMM arraca por encima del hardware

- Un programa que corre en una maquina virtual no tiene forma de saber que estan en una VM
- Va a correr identico a como si corriera en una maquina real
- Lo unico que podria alterarse es el tiempo de ejecucion (pero podria usarse tiempo virtual para esto)


### Que problemas tiene esto?

- Las maquinas virtuales van a consumir muchos recursos
- El hardware subyacente deberia de proveer suficientes recursos para iniciar con esto
- El cambio de contexto seria caro, pero es parecido a un sistema multiprogramado. Siempre es un problema pero es comparable con multiprogramación
- El scheduler en VMM es simple, se hace Round Robin. 
- El hardware verdadero tendrá ciertas caracteristicas y sobre eso es que se construyen "mundos virtuales" para las máquinas.
- Todo lo que tiene el hardware verdadero lo tiene el virtual. El VMM recibe las interrupciones reales y luego se las pasa a los manejadores de interrupciones virtuales en las VMs
- Los sistemas operativos por sobre la VMMs son "completos", ellos creen que estan ejecutando en modo privilegiado. 
- Nada es simulado sino virtualizado, un programa en una VM corre en el hardware real


Existe una contradicción cuando se diseña un VMM porque se 

siempre que corre el VMM el hardware eta en modo privilegiado, necesita tener acceso a todo
siempre que este corriendo una maquina virtual el hardware verdadero tiene que estar en modo NO privilegiado
- Hay que asegurar la virtualización
- El VM es un programa más.

Imaginemono un byte el cual nos dice el modo del hardware (1- privilegiado) (0 - no privilegiado)


Programa de aplicacion
|
SO
Hardware virtual' / bit con el modo
|
VMM
Hardware verdadero / bit con el modo

cuando corre el programa de aplicacion el hardware real está en modo no privilegiado y el hardware virtual esta en modo no privilegiado. 

Supongamos que la aplicacion hace una division por cero: 
- que hace el hardware real? dispara un _trap_ y el mecanismo especializo en atender el trap. Todo lo que ocurre en el hardware real despiera al VMM. El hardware real se pone en modo privilegiado. 

Cuando un error como division por cero se da, el VMM tiene forma de saber cual VM estaba corriendo en ese momento y le pasa el control al hardware virtual. Leugo se cambia a modo no privilegiado 

El hardware virtual se pone en modo privilegiado y el SO maneja el _trap_. El SO de la máquina virtual corre la rutina en el hardware real la cual en este momento está en modo no privilegiado, pero el hardware virtual está en modo privilegiado. El SO cree que esta ejecutando una operacion privilegiada. Luego de manejar el trap el bit de modo privilegiado en el hardware virtual se degrada a 0.

Que le va a pasar al programa que hace la division por cero?
No sabemos, eso lo decide el SO que corre en la VM, el VMM no está consciente de esto. 






Si en una VM en donde el hardware virtual esta en modo privilegiado y se intenta ejecutar una instrucción privilegida cuando el hardware real esta en modo NO privilegiado, se despierta el VMM, se pone en modo privilegiado, y en lugar de pasar el control a la VM se hace un mapeo de la instruccion privilegiada en el hardware real, se simula el comportamiento. El VMM es el encargado de hacer este mapeo/engaño. De esta forma el mundo virtual de donde viene esta instrucción se normaliza. Luego se devuelve el control a la VM y el hardware real se pone en modo NO privilegiado.



El concepto de VMM lo introdujo IBM. VM-370
Buscar alguna histora sobre VM-370 y CMS


Las maquinas no tienen forma de saber que son reales, este enfoque se llama maquinas virtuales puras

Una maquina de pronto tiene forma de darse cuenta de que no son real, este enfoque se llama maquinas virtuales impuras

Son trucos "sucios", poner algo en un registro para comunicarle a la VM (q se de cuenta) que es no real. 

Para que queremos esto? 
el enfoque de VM tiene a ser ineficiente. Hay muchos mapeos
Si se sabe que no es real entonces se pueden ahorrar estos mapeos 
Las maquinas virtuales impuras buscan estos para comunicarse con el monitor y optimiazar cosas y ser más eficientes. 

VM-370 es un esquema de maquina virtual impuras.

Paravirtualizacion, el hardware virtualizado es un subconjunto del hardware 

Paravirtualización. Es un subconjunto del hardware
Ejemplo procesador 386 corria varias VMs 8086

Para que la virtualizcion funcione
Cualquiere instruccions peligrosa, que ponga en riesto la virtualizacion, tiene que ser privilegiada. Para poner el hardware verdadero en modo no privilegiado para que llegue el VMM y arregle esto. El 386 no cumplia esto. Habian ciertas instrucciones que eran privilegiadas y que no eran tomadas por el VMM.


Tiempo virtual: 


Ventajas

- cada maquina esta totalmente aislada de las otras
- se puede correr cualquier programa
- proteccion del sistema operativo
- provisionamiento de software


Desventajas
- Son mas lentas
- hay que tener muchos recursos para tener varias VMs
- 

























