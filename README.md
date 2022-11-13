Laboratorio 3: Scheduler
========================

- En `proc.h` agregamos el atributo `int priority` dentro de `struct proc`.
  A número más bajo, prioridad más alta.

- En `proc.h` agregamos la constante `MAX_PRIORITY` que determina la cantidad de
  buffers o colas de prioridad de procesos con estado RUNNABLE.
  
- Se agrego en proc.c la funcion not_emtyqeuePriority(void), que ve si no estan vacias las colas de prioridad.


- Agregamos la entrada en `Makefile` para compilar `cbuffer.c` con su header.

- Integramos los programas de usuario `frutaflops` y `verduiops` agregando las
  entradas correspondientes en `Makefile`.



- Funciones modificadas en `proc.c`:
    - `void pinit(void)`
    - `void userinit(void)`
    - `int fork(void)`
    - `void scheduler(void)`
    - `void yield(void)`
    - `static void wakeup1(void *chan)`
    - `void sleep(void *chan, struct spinlock *lk)`
    - `int kill(int pid)`
    - `void procdump(void)`

Explicación de lo que hace el scheduler original en cada transcición de estado de un proceso:
---------------------------------------------------------------------------------------------

1. Hacer que un proceso que duerme sea planificable:
    - El estado del proceso es SLEEPING. Se llama
      `proc.c:void wakeup (void *chan)` que a su vez llama a
      `proc.c:static void wakeup1 (void *chan)` cambiando el estado a RUNNABLE.

2. Ejecutar un proceso planificable:
    - La función `proc.c:void scheduler(void)` busca en un arreglo (en donde
      están todos los procesos) el primer proceso con estado RUNNABLE. Lo
      cambia a RUNNING y hace el cambio de contexto para su ejecución.

3. Replanificar un proceso que ha agotado su tiempo de procesador disponible:
    - Si el proceso está RUNNING y todavía no finalizó su tarea, en la
      siguiente interrupción, `trap.c:void trap(struct trapframe *tf)` llama a
      `proc.c:void yield(void)` que cambia su estado a RUNNABLE y llama a
      `proc.c:void sched(void)` para luego entrar en
      `proc.c:void scheduler(void)` y ejecutar otro proceso.

4. Bloquear un proceso que está esperando entrada/salida:
    - Cambia su estado a SLEEPING, por medio de la función
      `proc.c:void sleep(void *chan, struct spinlock *lk)`.

Explicación de lo que hace el nuevo scheduler para los mismos cambios de estado:
--------------------------------------------------------------------------------

1. Hacer que un proceso que duerme sea planificable:
    - Igual que antes, solo que además
      `proc.c:static void wakeup1 (void *chan)` incrementa la prioridad del
      proceso y lo encola en el buffer correspondiente.

2. Ejecutar un proceso planificable:
    - La función `proc.c:void scheduler(void)` Considera los buffers (en donde
      están todos los procesos con estado RUNNABLE). Toma del buffer de mayor
      prioridad el primer proceso. Lo cambia a RUNNING y hace el cambio de
      contexto para su ejecución. Si el buffer está vacío pasa al siguiente.

3. Replanificar un proceso que ha agotado su tiempo de procesador disponible:
    - Si el proceso está RUNNING y todavía no finalizó su tarea, en la
      interrupción correspondiente (depende de su prioridad)
      `trap.c:void trap(struct trapframe *tf)` llama a
      `proc.c:void yield(void)` que cambia su estado a RUNNABLE, incrementa su
      prioridad y llama a `proc.c:void sched(void)` para luego entrar en
      `proc.c:void scheduler(void)` y ejecutar otro proceso.

4. Bloquear un proceso que está esperando entrada/salida:
    - Igual que antes, pero se decrementa su prioridad .

