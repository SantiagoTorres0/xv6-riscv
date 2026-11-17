# Exploración de la Planificación de Procesos en XV6

* **Curso:** Sistemas Operativos
* **Integrantes:**
  * Santiago Torres Martínez - 202521423
  * Lissete Johana Patiño Quinayas - 202520977
* **Repositorio Fork en GitHub:** https://github.com/SantiagoTorres0/xv6-riscv

---

## 1. Preparación del Repositorio y del Entorno

### 1.1 Instrucciones Reproducibles de Configuración
Para replicar el entorno de trabajo y ejecución del sistema operativo XV6 para arquitectura RISC-V, se realizaron los siguientes pasos:

1. Se creó un *fork* del repositorio oficial `mit-pdos/xv6-riscv` en GitHub.
2. Se clonó el repositorio localmente en un directorio sin espacios en blanco en su ruta (para evitar errores con el parámetro `-ffile-prefix-map` de GCC durante la compilación):
   ```bash
   git clone https://github.com/SantiagoTorres0/xv6-riscv
   cd xv6-riscv
   ```
3. Se instalaron las herramientas de compilación cruzada para RISC-V y el emulador QEMU:
   ```bash
   sudo apt update
   sudo apt install -y git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
   ```

### 1.2 Información del Sistema Operativo Anfitrión y Commit Base
* **Versión de Linux donde se clonó el código fuente:**
  ```text
  Linux santiago-HP-Laptop 6.17.0-22-generic #22~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Tue Mar 17 15:40:14 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
  PRETTY_NAME="Ubuntu 24.04.3 LTS"
  ```
* **Identificador (hash) del commit inicial desde el cual se comenzó a trabajar:**
  ```text
  fa2ddcb7ca87a77af579380a6eafd5aff163b44f
  ```

### 1.3 Herramientas de Compilación y Ejecución
Para compilar y emular XV6 para RISC-V se utilizaron las siguientes herramientas (compilador cruzado `riscv64-linux-gnu-gcc`, enlazador `riscv64-linux-gnu-ld`, emulador `qemu-system-riscv64`, depurador `gdb-multiarch` y `make`):

![Herramientas usadas y sus versiones](img/versiones-herramientas.png)



---

## 2. Compilación y Ejecución de XV6

### 2.1 Evidencia de XV6 Operativo
Se compiló el código fuente del kernel y las utilidades de usuario mediante `make qemu`. Una vez iniciado el shell (`$ `), se ejecutaron los comandos `ls` y `cat README` para verificar que el sistema de archivos y la ejecución de procesos de usuario operan correctamente:

```text
xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ cat README
xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
Version 6 (v6). xv6 loosely follows the structure and style of v6,
but is implemented for a modern RISC-V multiprocessor using ANSI C.

ACKNOWLEDGMENTS
...
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2227
cat            2 3 32864
echo           2 4 31720
forktest       2 5 15856
grep           2 6 36240
init           2 7 32216
kill           2 8 31680
ln             2 9 31504
ls             2 10 34808
mkdir          2 11 31736
rm             2 12 31720
sh             2 13 54168
stressfs       2 14 32608
usertests      2 15 178800
grind          2 16 47536
wc             2 17 33816
zombie         2 18 31176
console        3 19 0
```

* **Descripción de la evidencia:** La salida anterior muestra la secuencia completa de arranque del kernel (`xv6 kernel is booting`), la inicialización de los núcleos virtuales por defecto (`hart 1 starting` y `hart 2 starting`), el lanzamiento del proceso inicial de consola (`init: starting sh`) y la correcta lectura del archivo `README` junto con el listado de binarios ejecutables disponibles en el sistema de archivos.

### 2.2 Configuración y Ejecución con una Sola CPU Virtual
Para estudiar la selección de procesos sobre una única CPU sin modificar la configuración de hardware del equipo anfitrión, se ejecutó XV6 pasando el parámetro `CPUS=1` al comando `make` (el cual sobreescribe la variable `CPUS := 3` definida por defecto en el `Makefile`):

```bash
make clean
make qemu CPUS=1
```

![Arranque de XV6 con una sola CPU virtual](img/evidencia-arranque-1-cpu.png)

* **Evidencia de una sola CPU activa:** En la imagen se observa que, al ejecutar `make qemu CPUS=1`, el kernel imprime `xv6 kernel is booting` y pasa directamente a `init: starting sh`, omitiendo los mensajes `hart 1 starting` y `hart 2 starting` que sí aparecían en el arranque estándar con 3 CPUs. Esto confirma que QEMU inicializó únicamente el *hart* 0 (una sola CPU virtual).

---

## 3. Investigación del Planificador Original de XV6

*(Sección a completar con el análisis del Integrante B)*

### 3.1 Algoritmo o Política de Planificación por Defecto
* **Explicación en palabras propias:** [Explicar aquí la política Round-Robin sin prioridades ni envejecimiento que utiliza XV6 por defecto y cómo se evidencia en el comportamiento del sistema].
* **Evidencia:** [Incluir explicación o prueba de comportamiento observada].

### 3.2 Archivo y Función de Selección del Siguiente Proceso
* **Archivo y función:** `kernel/proc.c` — función `scheduler(void)`.
* **Explicación y fragmento de código:**
  ```c
  // Pegar aquí el fragmento relevante de scheduler() en kernel/proc.c
  ```

### 3.3 Relación con los Estados de los Procesos y Eventos de Cambio de Contexto
* **Estados de proceso (`kernel/proc.h`):** Explicar `enum procstate` (`UNUSED`, `USED`, `SLEEPING`, `RUNNABLE`, `RUNNING`, `ZOMBIE`) y los campos relevantes de `struct proc`.
* **Eventos que ceden o recuperan la CPU (`kernel/proc.c` y `kernel/trap.c`):** Explicar el ciclo entre `usertrap()`/`kerneltrap()` (interrupción de temporizador), `yield()`, `sched()` y `swtch()`.
* **Fragmentos de código de respaldo:**
  ```c
  // Pegar aquí los fragmentos relevantes de kernel/proc.h, kernel/proc.c y kernel/trap.c
  ```

### 3.4 Incorporación del Planificador durante la Compilación y Construcción
* **Proceso de construcción (`Makefile`):** Explicar cómo `kernel/proc.c` y `kernel/trap.c` son compilados a archivos objeto (`kernel/proc.o`, `kernel/trap.o` listados en la variable `OBJS`) y enlazados mediante `riscv64-linux-gnu-ld` usando `kernel/kernel.ld` para generar el binario final `kernel/kernel`.
* **Fragmento del `Makefile`:**
  ```makefile
  # Pegar aquí el fragmento de OBJS y la regla de construcción de $K/kernel del Makefile
  ```

---

## 4. Descripción de MLFQ y Exploración de su Incorporación en XV6

### 4.1 Descripción Inicial del Algoritmo MLFQ y Dudas Identificadas
*(Redactada antes de iniciar el diálogo socrático con la IA)*
* **Descripción inicial:** MLFQ es un planificador de procesos que queremos analizar para incorporar en el sistema operativo XV6. Se basa en varias colas de procesos organizadas por niveles de prioridad, en las que al proceso con mayor prioridad se le asigna un tiempo de ejecución (quantum). Al agotar ese tiempo de ejecución, el proceso pierde prioridad (baja a una cola inferior) y da paso al siguiente proceso. Además, cada proceso nuevo que entra al sistema lo hace con la prioridad más alta.
* **Dudas identificadas:** 
  1. ¿Cómo funciona exactamente Round Robin dentro de las colas de MLFQ?

  2. Si entran procesos nuevos o interactivos constantemente en la cola más alta, ¿en qué momento se ejecutan los procesos que ya agotaron su tiempo de ejecución y bajaron de prioridad sin haber terminado?

  3. ¿Cómo se mide en XV6 el tiempo que lleva ejecutándose un proceso y cuántas colas o qué tamaño de quantum deberíamos usar?

### 4.2 Prompt 1 Utilizado para la Interacción Socrática
```text
[Pegar aquí exactamente el Prompt 1 completo enviado a la IA, incluyendo la descripción inicial de MLFQ y la síntesis del planificador original de XV6]
```

### 4.3 Descripción Revisada y Coherente de MLFQ para XV6
* [Redactar la versión refinada del algoritmo tras la interacción socrática, definiendo la variante específica y los parámetros propuestos].

### 4.4 Propuesta de Cambios Necesarios en XV6
*(Sin código de implementación; justificando qué necesidad atiende cada cambio y qué evidencia del código lo respalda)*

1. **Estructuras de datos de procesos (`kernel/proc.h` - `struct proc`):**
   * **Cambio propuesto:** [Ej. Agregar campos para prioridad actual, ticks consumidos en el nivel actual, etc.].
   * **Necesidad que atiende y justificación:** [Explicar].
2. **Inicialización y reinicio de prioridades (`kernel/proc.c` - `allocproc()`, `userinit()`, `fork()`):**
   * **Cambio propuesto:** [Explicar].
   * **Necesidad que atiende y justificación:** [Explicar].
3. **Contabilidad de tiempo y degradación de prioridad (`kernel/trap.c` - `usertrap()`, `kernel/proc.c` - `yield()`):**
   * **Cambio propuesto:** [Explicar].
   * **Necesidad que atiende y justificación:** [Explicar].
4. **Selección por prioridad y *Priority Boost* (`kernel/proc.c` - `scheduler()` / `clockintr()`):**
   * **Cambio propuesto:** [Explicar].
   * **Necesidad que atiende y justificación:** [Explicar].

### 4.5 Decisiones Pendientes de Investigación o Soporte Adicional
* [Listar aspectos técnicos que requerirían validación experimental o investigación adicional antes de implementar, como sincronización con *spinlocks* (`p->lock`), elección óptima del periodo $S$ para evitar inanición sin degradar la interactividad, y manejo de procesos que hacen *sleep* frecuente].

### 4.6 Escenarios de Comprobación Teórica
*(Descripción conceptual sin programación ni ejecución)*

* **Escenario 1 (Regla 3 y Regla 4 - Degradación de procesos CPU-bound):**
  * **Procesos participantes:** [Describir].
  * **Acciones de los procesos:** [Describir].
  * **Regla que comprueba:** [Describir].
  * **Comportamiento esperado:** [Describir].
* **Escenario 2 (Regla 1 y Regla 2 - Prioridad de procesos interactivos I/O-bound frente a CPU-bound):**
  * **Procesos participantes:** [Describir].
  * **Acciones de los procesos:** [Describir].
  * **Regla que comprueba:** [Describir].
  * **Comportamiento esperado:** [Describir].
* **Escenario 3 (Regla 5 - *Priority Boost* y prevención de inanición):**
  * **Procesos participantes:** [Describir].
  * **Acciones de los procesos:** [Describir].
  * **Regla que comprueba:** [Describir].
  * **Comportamiento esperado:** [Describir].

---

## 5. Evidencia del Aprendizaje con IA y Conclusiones

### 5.1 Identificación de la Herramienta de IA
* **Servicio y modelo utilizado:** [Ej. Google Gemini / ChatGPT / etc., indicando versión si está disponible].

### 5.2 Fragmentos Seleccionados del Diálogo Socrático
*(Incluir de 3 a 5 intercambios significativos que muestren hipótesis, comprobaciones o correcciones)*

1. **Intercambio 1:**
   * **Pregunta de la IA:** ...
   * **Respuesta de los estudiantes:** ...
2. **Intercambio 2:**
   * **Pregunta de la IA:** ...
   * **Respuesta de los estudiantes:** ...
3. **Intercambio 3:**
   * **Pregunta de la IA:** ...
   * **Respuesta de los estudiantes:** ...

### 5.3 Conclusiones Generales y Retroalimentación de la IA
* **Conclusiones del equipo:** [Redactar conclusiones sobre las diferencias entre Round-Robin en XV6 y MLFQ, y las implicaciones arquitectónicas de incorporarlo].
* **Retroalimentación obtenida:** [Resumir la validación final de la IA sobre las conclusiones].

### 5.4 Reflexiones Individuales

* **Reflexión de Santiago [Apellido]:**
  > [Escribir un párrafo individual indicando qué comprendió, qué aportó al trabajo y qué cuestión aún necesita estudiar].

* **Reflexión de [Nombre del Compañero]:**
  > [Escribir un párrafo individual indicando qué comprendió, qué aportó al trabajo y qué cuestión aún necesita estudiar].