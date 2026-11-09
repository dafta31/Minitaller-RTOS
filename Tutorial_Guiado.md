# 🚀 Tutorial Guiado: Construir el Cronograma con FreeRTOS™ y Modificar Prioridades, Duración y Periodo

**Duración Estimada:** 30 minutos  
**Modalidad:** Trabajo individual en la terminal de Ubuntu (o WSL2)  
**Propósito Práctico:** Descargar el kernel de FreeRTOS, estructurar un proyecto mínimo, compilar usando el puerto POSIX y experimentar con la modificación de parámetros de tareas (prioridades, duración de procesamiento y periodo de activación) para analizar visualmente el cronograma de ejecución (*scheduling chart*).

---

## 📖 1. Introducción

En los **Sistemas Operativos de Tiempo Real (RTOS)** como FreeRTOS™, la gestión eficiente del procesador es fundamental para garantizar que las tareas críticas respondan dentro de sus plazos temporales (*deadlines*). A diferencia de los sistemas operativos de propósito general (GPOS), FreeRTOS emplea un planificador preventivo por prioridades fijas (*Fixed-Priority Preemptive Scheduler*).

Este tutorial guiado permitirá experimentar de forma práctica con el Kernel de FreeRTOS en un entorno emulado en Linux utilizando el **puerto POSIX**. A través de la construcción y modificación de una aplicación con múltiples tareas (`vControlTask`, `vKeyHandlerTask`, `vLogTask` e `IdleTask`), los estudiantes podrán observar directamente cómo la alteración de parámetros clave como la **prioridad**, la **duración de cómputo** y el **periodo de activación** impacta la asignación de CPU y el comportamiento del sistema.

---

## 🎯 2. Objetivos del Tutorial

Al finalizar este laboratorio, el estudiante será capaz de:

1. **Estructurar un Proyecto Mínimo con FreeRTOS™**: Descargar e integrar los archivos esenciales del Kernel (`tasks.c`, `queue.c`, `list.c`, `heap_4.c` y el puerto `Posix`).
2. **Configurar el Comportamiento del Kernel**: Entender y personalizar las directivas clave en `FreeRTOSConfig.h` (tales como `configTICK_RATE_HZ`, `configUSE_PREEMPTION` y `configUSE_TIME_SLICING`).
3. **Analizar la Multitarea Preventiva (*Preemption*)**: Observar el comportamiento del planificador determinista al modificar la prioridad de ejecución de las tareas.
4. **Comprender la Relación Periodo-Duración**: Evaluar experimentalmente cómo la frecuencia de activación (*periodo*) y el tiempo de ocupación de la CPU (*duración*) influyen en la latencia de respuesta y en el tiempo de inactividad de la CPU (`IdleTask`).
5. **Desarrollar Habilidades de Compilación y Depuración**: Compilar aplicaciones multitarea en C usando `gcc` con soporte para hilos POSIX (`-pthread`) y solucionar problemas comunes de desarrollo en entorno Linux.

---

## 🧰 3. Materiales, Componentes y Software Necesarios

Para llevar a cabo este tutorial guiado no se requiere hardware embebido físico, ya que toda la simulación se ejecuta directamente sobre el Kernel de Linux mediante el puerto POSIX de FreeRTOS.

### **Software Requerido**
* **Sistema Operativo**: Ubuntu Linux 20.04 / 22.04 LTS (o WSL2 en Windows).
* **Compilador de C**: `gcc` (GNU Compiler Collection) con soporte para la librería de hilos POSIX (`pthread`).
* **Herramientas de Construcción y Control de Versiones**: `make` y `git`.
* **Editor de Texto en Terminal**: `nano` (o cualquier editor de texto plano).

### **Componentes del Proyecto y Código Fuente**
* **FreeRTOS Kernel**: Repositorio oficial (`FreeRTOS-Kernel.git`) que contiene:
  * `tasks.c`: Núcleo del planificador de tareas y gestión de estados.
  * `queue.c`: Primitivas de comunicación y sincronización (colas IPC).
  * `list.c`: Gestión de estructuras de datos internas (listas de estado).
  * `portable/MemMang/heap_4.c`: Administrador de asignación dinámica de memoria.
  * `portable/ThirdParty/GCC/Posix`: Capa de abstracción para adaptar el Kernel de FreeRTOS a hilos de Linux.
* **Archivos del Proyecto del Estudiante**:
  * `config/FreeRTOSConfig.h`: Archivo de cabecera con la configuración del Kernel.
  * `src/main.c`: Programa principal con la lógica de tareas, cola de teclado y monitor del cronograma.

---

## 📝 Guía Rápida del Editor `nano`

* **Ctrl + W**: Busca texto. Escriban el texto en la línea *Buscar:* de abajo y presionen `Enter`. El cursor saltará automáticamente a esa línea.
* **Flechas de dirección**: Mueven el cursor por el archivo. `Backspace` borra el carácter a la izquierda.
* **Ctrl + O** y luego `Enter`: Guardan los cambios realizados en el archivo.
* **Ctrl + X**: Salen del editor `nano`.

---

## 🛠️ Paso a Paso del Laboratorio

### Paso 0: Instalación de Herramientas
Ejecuten el siguiente comando en la terminal para asegurarse de contar con todos los paquetes necesarios:

```bash
sudo apt update && sudo apt install -y git build-essential gcc nano
```

---

### Paso 1: Descargar el Kernel de FreeRTOS
```bash
mkdir -p ~/Taller-FreeRTOS && cd ~/Taller-FreeRTOS
git clone --depth 1 https://github.com/FreeRTOS/FreeRTOS-Kernel.git
ls FreeRTOS-Kernel
```
**Checkpoint:** Aparecen `tasks.c`, `queue.c`, `list.c`, `include` y `portable`.

---

### Paso 2: Localizar los Archivos Mínimos que Exige FreeRTOS
Un proyecto de FreeRTOS necesita como mínimo estos archivos:
* `tasks.c` (planificador y tareas)
* `queue.c` (colas, que usaremos para las pulsaciones de tecla)
* `list.c` (listas internas del kernel)
* `port.c` del puerto de su plataforma (aquí usaremos el puerto POSIX, que corre en Linux)
* Un archivo `heap_x.c` (en este taller usaremos `heap_4.c`)

Verifiquen que existen ejecutando:
```bash
cd ~/Taller-FreeRTOS/FreeRTOS-Kernel
ls tasks.c queue.c list.c
ls portable/MemMang/heap_4.c
find portable -path "*Posix*" -name "port.c"
find portable -path "*Posix*" -name "wait_for_event.c"
```
**Checkpoint:** Los dos últimos comandos `find` muestran una ruta. Si es distinta de `portable/ThirdParty/GCC/Posix`, anótenla: la usarán en el comando de compilación.

---

### Paso 3: Crear la Estructura del Proyecto
```bash
cd ~/Taller-FreeRTOS
mkdir -p config src
```
Después de este paso, la estructura de la carpeta queda así:
```text
Taller-FreeRTOS/
├── FreeRTOS-Kernel/   (descargado en el Paso 1)
├── config/            (ustedes escriben FreeRTOSConfig.h)
└── src/               (ustedes escriben main.c)
```

---

### Paso 4: Crear el Archivo de Configuración (`FreeRTOSConfig.h`)
```bash
nano config/FreeRTOSConfig.h
```
Escriban este contenido dentro del archivo:

```c
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

#define configUSE_PREEMPTION              1
#define configUSE_TIME_SLICING            1
#define configUSE_IDLE_HOOK               0
#define configUSE_TICK_HOOK               0
#define configTICK_RATE_HZ                ((TickType_t)1)
#define configMAX_PRIORITIES              7
#define configMINIMAL_STACK_SIZE          ((unsigned short)256)
#define configTOTAL_HEAP_SIZE             ((size_t)(64 * 1024))
#define configMAX_TASK_NAME_LEN           16
#define configUSE_16_BIT_TICKS            0
#define configCHECK_FOR_STACK_OVERFLOW    0
#define configASSERT(x)                   if((x) == 0) { for(;;); }
#define INCLUDE_vTaskDelay                1
#define INCLUDE_vTaskDelayUntil           1
#define INCLUDE_vTaskSuspend              1

#endif
```

**Lo más importante de este archivo:**
* `configTICK_RATE_HZ` en `1` hace que un tick dure 1 segundo, igual que en el cronograma del taller.
* `configUSE_PREEMPTION` en `1` permite que una tarea de mayor prioridad interrumpa a otra.
* `configUSE_TIME_SLICING` en `1` hace que las tareas de igual prioridad se turnen el uso de la CPU.
* `configMAX_PRIORITIES` en `7` permite prioridades numeradas de 0 a 6.

Presionen `Ctrl+O`, `Enter` y `Ctrl+X` para guardar y salir.

---

### Paso 5: Crear el Programa Principal (`src/main.c`)
```bash
nano src/main.c
```
Escriban este contenido completo:

```c
/*
 * main.c - Cronograma con FreeRTOS real (puerto POSIX).
 * Las tareas se configuran en la seccion CONFIGURACION.
 */
#include <stdint.h>
#include <stdio.h>
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"

/* ================== CONFIGURACION ================== */
#define PRIO_CONTROL      3    /* vControlTask */
#define PERIODO_CONTROL   4    /* ticks entre activaciones */
#define DUR_CONTROL       1    /* ticks de CPU por activacion */

#define PRIO_TECLA        2    /* vKeyHandlerTask */
#define DUR_TECLA         2    /* ticks de CPU por pulsacion */

#define PRIO_LOG          1    /* vLogTask */
#define PERIODO_LOG       2
#define DUR_LOG           1

#define PRIO_FUENTE       5    /* simula la interrupcion del teclado */
#define PRIO_MONITOR      6    /* dibuja el cronograma; debe ser la mayor */

/* Ticks en los que se presiona una tecla */
static const TickType_t teclas_en[] = {1, 4, 6, 7, 7};
#define N_TECLAS  (sizeof(teclas_en) / sizeof(teclas_en[0]))
/* =================================================== */

#define N_APP        3
#define IDX_CONTROL  0
#define IDX_TECLA    1
#define IDX_LOG      2
#define MAX_T        60
#define VENTANA      30

static const char *nombres[N_APP]    = {"vControlTask", "vKeyHandlerTask", "vLogTask"};
static const int   periodos[N_APP]   = {PERIODO_CONTROL, 0, PERIODO_LOG};
static const int   duraciones[N_APP] = {DUR_CONTROL, DUR_TECLA, DUR_LOG};
static const char *colores[N_APP]    = {"\033[31m", "\033[34m", "\033[35m"};

/* Registro hecho por las tareas en el tick real en que ocurre cada cosa */
static volatile char corre[MAX_T + 1][N_APP];   /* 'R' si la tarea tiene la CPU en ese tick */
static volatile char espera[MAX_T + 1][N_APP];  /* 1 si la tarea esta lista pero sin CPU */
static volatile char llega[MAX_T + 1][N_APP];   /* 1 si llega una activacion en ese tick */
static QueueHandle_t xColaTeclas;

/* Marca la espera entre el tick en que quedo lista y el tick en que empezo a correr */
static void registrar_espera(int i, TickType_t liberada, TickType_t inicio) {
    for (TickType_t t = liberada; t < inicio && t <= MAX_T; t++) {
        espera[t][i] = 1;
    }
}

/* Ocupa la CPU durante 'ticks' ticks y marca cada tick en que corre */
static void hacer_trabajo(int i, TickType_t liberada, int ticks) {
    TickType_t inicio = xTaskGetTickCount();
    registrar_espera(i, liberada, inicio);
    for (int k = 0; k < ticks; k++) {
        TickType_t ahora = xTaskGetTickCount();
        if (ahora <= MAX_T) corre[ahora][i] = 'R';
        while (xTaskGetTickCount() == ahora) { }
    }
}

static void vTareaPeriodica(void *pv) {
    int i = (int)(intptr_t)pv;
    TickType_t ultimo = xTaskGetTickCount();
    for (;;) {
        vTaskDelayUntil(&ultimo, periodos[i]);   /* espera la siguiente activacion */
        TickType_t liberada = ultimo;
        if (liberada <= MAX_T) llega[liberada][i] = 1;
        hacer_trabajo(i, liberada, duraciones[i]);
    }
}

static void vKeyHandlerTask(void *pv) {
    TickType_t liberada;
    (void)pv;
    for (;;) {
        xQueueReceive(xColaTeclas, &liberada, portMAX_DELAY);
        hacer_trabajo(IDX_TECLA, liberada, DUR_TECLA);
    }
}

static void vFuenteTeclado(void *pv) {
    (void)pv;
    for (unsigned int k = 0; k < N_TECLAS; k++) {
        TickType_t ahora = xTaskGetTickCount();
        if (teclas_en[k] > ahora) vTaskDelay(teclas_en[k] - ahora);
        TickType_t liberada = xTaskGetTickCount();
        if (liberada <= MAX_T) llega[liberada][IDX_TECLA] = 1;
        xQueueSend(xColaTeclas, &liberada, 0);
    }
    vTaskSuspend(NULL);
}

static void dibujar(int t) {
    int inicio = (t > VENTANA) ? t - VENTANA + 1 : 1;

    printf("\033[H\033[2J");
    printf("=== CRONOGRAMA CON FREERTOS REAL (puerto POSIX) ===\n");
    printf("Tick cada 1 s. Mostrando hasta el tick %d\n\n", t);

    printf("  %-16s ", "eventos");
    for (int x = inicio; x <= t; x++) {
        int n = 0, quien = -1;
        for (int i = 0; i < N_APP; i++) {
            if (llega[x][i]) { n++; quien = i; }
        }
        if (n == 0)      printf("  ");
        else if (n > 1) printf("* ");
        else            printf("%d ", quien + 1);
    }
    printf("\n");

    printf("  %-16s ", "tick");
    for (int x = inicio; x <= t; x++) printf("%2d", x % 10);
    printf("\n");

    for (int i = 0; i < N_APP; i++) {
        printf("  %s%-16s\033[0m ", colores[i], nombres[i]);
        for (int x = inicio; x <= t; x++) {
            if (corre[x][i] == 'R')  printf("%s██\033[0m", colores[i]);
            else if (espera[x][i])   printf("%s░░\033[0m", colores[i]);
            else                     printf("  ");
        }
        printf("\n");
    }

    printf("  %-16s ", "IdleTask");
    for (int x = inicio; x <= t; x++) {
        int alguna = 0;
        for (int i = 0; i < N_APP; i++) if (corre[x][i] == 'R') alguna = 1;
        printf("%s", alguna ? "  " : "\033[32m██\033[0m");
    }
    printf("\n\n");

    printf("  Estado en el tick %d:\n", t);
    for (int i = 0; i < N_APP; i++) {
        const char *txt = "BLOCKED";
        if (corre[t][i] == 'R')     txt = "RUNNING";
        else if (espera[t][i])      txt = "READY";
        printf("  %s%-16s\033[0m prioridad %d  ->  %s\n", colores[i], nombres[i],
               i == IDX_CONTROL ? PRIO_CONTROL : (i == IDX_TECLA ? PRIO_TECLA : PRIO_LOG), txt);
    }
    printf("\n  Leyenda: ██ RUNNING   ░░ READY   (vacio) BLOCKED\n");
    fflush(stdout);
}

static void vMonitor(void *pv) {
    TickType_t ultimo = xTaskGetTickCount();
    (void)pv;
    for (;;) {
        vTaskDelayUntil(&ultimo, 1);
        TickType_t ahora = xTaskGetTickCount();
        /* Se dibuja con dos ticks de retraso para que todos los registros ya esten escritos */
        if (ahora >= 3 && ahora - 2 <= MAX_T) dibujar((int)(ahora - 2));
    }
}

int main(void) {
    xColaTeclas = xQueueCreate(8, sizeof(TickType_t));

    xTaskCreate(vMonitor, "Monitor", configMINIMAL_STACK_SIZE * 2, NULL, PRIO_MONITOR, NULL);
    xTaskCreate(vFuenteTeclado, "Teclado", configMINIMAL_STACK_SIZE, NULL, PRIO_FUENTE, NULL);
    xTaskCreate(vTareaPeriodica, nombres[IDX_CONTROL], configMINIMAL_STACK_SIZE,
                (void *)(intptr_t)IDX_CONTROL, PRIO_CONTROL, NULL);
    xTaskCreate(vKeyHandlerTask, nombres[IDX_TECLA], configMINIMAL_STACK_SIZE, NULL, PRIO_TECLA, NULL);
    xTaskCreate(vTareaPeriodica, nombres[IDX_LOG], configMINIMAL_STACK_SIZE,
                (void *)(intptr_t)IDX_LOG, PRIO_LOG, NULL);

    vTaskStartScheduler();
    for (;;) { }
    return 0;
}
```

Presionen `Ctrl+O`, `Enter` y `Ctrl+X` para guardar y salir.

Ahora hagan una copia de respaldo fuera de `src`, para poder volver a esta versión limpia al final del taller:
```bash
cp src/main.c main_original.c
```

**Lo más importante de este archivo:**
* La sección `CONFIGURACION` es la única que van a editar a lo largo de los experimentos.
* `vTaskDelayUntil()` activa `vControlTask` y `vLogTask` cada cierto número de ticks. Ese número es el periodo.
* `hacer_trabajo()` ocupa la CPU durante la duración configurada y registra en qué ticks corre la tarea.
* Cuando una tarea queda lista pero no tiene la CPU, `registrar_espera()` marca esos ticks como `READY` (`░░`). Así la fila muestra la espera real del kernel, no una estimación.
* `vMonitor` dibuja el cronograma cada segundo. Tiene prioridad 6 para ejecutar y dibujar antes que las demás tareas.

---

### Paso 6: Compilar
Escriban este comando en la terminal para compilar el proyecto:

```bash
cd ~/Taller-FreeRTOS
gcc -Wall -Wextra -g -pthread -Iconfig -Isrc   -IFreeRTOS-Kernel/include   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils   src/main.c   FreeRTOS-Kernel/tasks.c   FreeRTOS-Kernel/queue.c   FreeRTOS-Kernel/list.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/port.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c   FreeRTOS-Kernel/portable/MemMang/heap_4.c   -o cronograma
```

Si el Paso 2 mostró otra ruta para el puerto POSIX, cámbienla en las tres líneas del comando que contienen `Posix`.  
**Checkpoint:** El comando termina sin errores y aparece el archivo ejecutable `cronograma`.

---

### Paso 7: Ejecutar y Observar
Ejecuten el programa generado:
```bash
./cronograma
```

**Ojo**El programa dibujará el cronograma en pantalla en tiempo real (un tick por segundo). Para detenerlo en cualquier momento, presionen `Ctrl+C`.

cronograma muestra tres símbolos por tarea en cada tick (cada segundo):

`██`: la tarea está usando el procesador.

`░░`: la tarea quiere usar el procesador, pero otra tarea lo tiene. Está esperando su turno.

Espacio vacío: la tarea no tiene nada que hacer en ese momento.

**Antes de dejarlo correr:**
* ¿En qué ticks debería activarse `vControlTask`? (Número `1` en la fila de eventos).
* ¿En qué ticks se presiona una tecla? (Número `2` en la fila de eventos).

Dejen correr la simulación hasta el tick 10 y comparen.

**¿Qué deben observar?**
* `vControlTask` tiene la mayor prioridad de la aplicación (3), así que en sus ticks de activación no debe aparecer `░░`.
* `vKeyHandlerTask` sí muestra `░░` cuando una pulsación debe esperar su turno. En el tick 7 llegan dos pulsaciones seguidas: la segunda espera hasta que termine la primera.

**Checkpoint:** Aparecen las cuatro filas (tres tareas e `IdleTask`), la fila de eventos y el estado de cada tarea en el tick actual.

---

### Paso 8: Cambiar la Prioridad de `vLogTask` de 1 a 2
Presionen `Ctrl+C` para detener el programa si sigue corriendo. Después abran el archivo con `nano`:

```bash
nano src/main.c
```

**Dentro de `nano`:**
1. Presionen `Ctrl+W`, escriban `#define PRIO_LOG` y presionen `Enter`. El cursor quedará en esta línea:
```c
#define PRIO_LOG          1    /* vLogTask */
```
2. Muevan el cursor hasta el número `1` y cámbienlo por `2`:
```c
#define PRIO_LOG          2    /* vLogTask */
```
3. Presionen `Ctrl+O`, `Enter` y `Ctrl+X` para guardar y salir.

Compilen y ejecuten de nuevo:
```bash
cd ~/Taller-FreeRTOS
gcc -Wall -Wextra -g -pthread -Iconfig -Isrc   -IFreeRTOS-Kernel/include   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils   src/main.c   FreeRTOS-Kernel/tasks.c   FreeRTOS-Kernel/queue.c   FreeRTOS-Kernel/list.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/port.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c   FreeRTOS-Kernel/portable/MemMang/heap_4.c   -o cronograma
./cronograma
```

**Qué deben observar:** Ahora `vLogTask` y `vKeyHandlerTask` tienen la misma prioridad (2). Cuando las dos están listas en el mismo tick, la CPU se turna entre ellas tick a tick (*Round-Robin / Time-Slicing*). La tarea que espera su turno muestra `░░`. Anoten en qué ticks ocurre. Presionen `Ctrl+C` para detener.

---

### Paso 9: Cambiar la Duración de `vControlTask` de 1 a 2 Ticks
Antes de este paso, regresen la prioridad de `vLogTask` a 1 para ver el efecto aislado. En `nano`, busquen `#define PRIO_LOG`, cambien el 2 por 1, guarden y salgan.

Ahora modifiquen la duración de control:
```bash
nano src/main.c
```

**Dentro de `nano`:**
1. Presionen `Ctrl+W`, escriban `#define DUR_CONTROL` y presionen `Enter`:
```c
#define DUR_CONTROL       1    /* ticks de CPU por activacion */
```
2. Cambien el `1` por `2`:
```c
#define DUR_CONTROL       2    /* ticks de CPU por activacion */
```
3. Presionen `Ctrl+O`, `Enter` y `Ctrl+X`.

Compilen y ejecuten:
```bash
gcc -Wall -Wextra -g -pthread -Iconfig -Isrc   -IFreeRTOS-Kernel/include   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils   src/main.c   FreeRTOS-Kernel/tasks.c   FreeRTOS-Kernel/queue.c   FreeRTOS-Kernel/list.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/port.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c   FreeRTOS-Kernel/portable/MemMang/heap_4.c   -o cronograma
./cronograma
```

**Predicción antes de ejecutar:** Cada vez que `vControlTask` se activa, ¿cuántos ticks seguidos ocupa la CPU?  
**Qué deben observar:** En la fila de `vControlTask`, cada activación ocupa dos ticks seguidos con `██`. Presionen `Ctrl+C` para detener.

---

### Paso 10: Cambiar el Periodo (Espaciamiento) de `vControlTask` de 4 a 3 Ticks
```bash
nano src/main.c
```

**Dentro de `nano`:**
1. Presionen `Ctrl+W`, escriban `#define PERIODO_CONTROL` y presionen `Enter`:
```c
#define PERIODO_CONTROL   4    /* ticks entre activaciones */
```
2. Cambien el `4` por `3`:
```c
#define PERIODO_CONTROL   3    /* ticks entre activaciones */
```
3. Presionen `Ctrl+O`, `Enter` y `Ctrl+X`.

Compilen y ejecuten:
```bash
gcc -Wall -Wextra -g -pthread -Iconfig -Isrc   -IFreeRTOS-Kernel/include   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils   src/main.c   FreeRTOS-Kernel/tasks.c   FreeRTOS-Kernel/queue.c   FreeRTOS-Kernel/list.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/port.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c   FreeRTOS-Kernel/portable/MemMang/heap_4.c   -o cronograma
./cronograma
```

**Predicción antes de ejecutar:** ¿En qué ticks se activa ahora `vControlTask`?  
**Qué deben observar:** La activación de `vControlTask` aparece en los ticks 3, 6 y 9. El número `1` de la fila de eventos marca cada activación. Presionen `Ctrl+C` para detener.

---

### Paso 11: Reto Práctico
Ahora cambien la prioridad de `vLogTask` de 1 a 4. Antes de compilar, escriban en un papel qué esperan que le suceda a `vLogTask`.

```bash
nano src/main.c
```
1. Presionen `Ctrl+W`, busquen `#define PRIO_LOG`, cambien el valor por `4`, guarden y salgan.
2. Compilen y ejecuten `./cronograma`. Comparen la salida con su predicción.

**Segunda parte del reto:**
Regresen `PRIO_LOG` a `2` y cambien `PRIO_TECLA` a `1`. En `nano`, usen `Ctrl+W` para ajustar ambas directivas, guarden y salgan.  
Compilen y ejecuten `./cronograma`. Presionen `Ctrl+C` después de unos ticks y observen la fila de `vKeyHandlerTask`.

**Pregunta del Reto:** ¿Qué pasa con las pulsaciones de tecla cuando `vKeyHandlerTask` tiene la prioridad más baja de la aplicación?

---

### Paso 12: Restaurar la Versión Original
Restauren el archivo guardado en el respaldo y compilen nuevamente:

```bash
cd ~/Taller-FreeRTOS
cp main_original.c src/main.c
gcc -Wall -Wextra -g -pthread -Iconfig -Isrc   -IFreeRTOS-Kernel/include   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix   -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils   src/main.c   FreeRTOS-Kernel/tasks.c   FreeRTOS-Kernel/queue.c   FreeRTOS-Kernel/list.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/port.c   FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c   FreeRTOS-Kernel/portable/MemMang/heap_4.c   -o cronograma
./cronograma
```
Para detener la ejecución presionen `Ctrl+C`.

---

### Paso 13: Relación con la Arquitectura de FreeRTOS
Estos son los archivos del kernel que utilizaron y la función que cumple cada uno en el sistema:
* `tasks.c`: Decide qué tarea ejecuta la CPU en cada tick según las prioridades asignadas.
* `queue.c`: Maneja la cola de comunicación de pulsaciones de teclado hacia `vKeyHandlerTask`.
* `list.c`: Almacena las estructuras de las tareas en listas internas de acuerdo con su estado (`READY`, `BLOCKED`, `SUSPENDED`).
* `port.c` (puerto POSIX): Adapta las tareas de FreeRTOS para que corran como hilos de Linux.
* `heap_4.c`: Administra la asignación dinámica de memoria cuando se invoca `xTaskCreate()`.
* `FreeRTOSConfig.h`: Define los parámetros globales y adapta el kernel a las necesidades del proyecto.

**Preguntas de Cierre:**
1. ¿Qué parámetro de la función `xTaskCreate()` corresponde a la prioridad de la tarea?
2. ¿Qué parámetro en `FreeRTOSConfig.h` define que la frecuencia de tick sea de 1 Hz (1 segundo)?
3. ¿Qué función utiliza `vControlTask` para esperar de forma periódica su siguiente activación?
4. Si `vControlTask` tuviera prioridad 6 y `vMonitor` también tuviera prioridad 6, ¿qué sucedería con la visualización del cronograma?

---

## ⚠️ 5. Resolución de Problemas (Troubleshooting)

A continuación se presentan los errores más frecuentes detectados durante el desarrollo del taller y cómo solucionarlos:

### 1. 🛑 Modificar `src/main.c` con `nano` pero no ver ningún cambio al ejecutar `./cronograma`
* **Causa**: Al editar el archivo fuente `src/main.c` con `nano` y guardarlo (Ctrl+O, Enter), las modificaciones se guardan únicamente como texto plano en el disco. El archivo binario ejecutable (`cronograma`) no se actualiza automáticamente.
* **Solución**: **SIEMPRE debes volver a ejecutar el comando de compilación completo y luego ejecutar el binario** después de realizar y guardar cualquier cambio en `src/main.c`:

```bash
cd ~/Taller-FreeRTOS
gcc -Wall -Wextra -g -pthread -Iconfig -Isrc \
  -IFreeRTOS-Kernel/include \
  -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix \
  -IFreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils \
  src/main.c \
  FreeRTOS-Kernel/tasks.c \
  FreeRTOS-Kernel/queue.c \
  FreeRTOS-Kernel/list.c \
  FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/port.c \
  FreeRTOS-Kernel/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c \
  FreeRTOS-Kernel/portable/MemMang/heap_4.c \
  -o cronograma

./cronograma
```
Si ejecutas `./cronograma` directamente sin recompilar con este comando, estarás ejecutando la versión antigua sin tus cambios.

### 2. ❌ Error de compilación: `fatal error: FreeRTOS.h: No such file or directory`
* **Causa**: El compilador `gcc` no encuentra los archivos de cabecera porque falta una bandera de inclusión (`-I`) o la terminal no está posicionada en el directorio base `~/Taller-FreeRTOS`.
* **Solución**: Asegúrate de estar en `cd ~/Taller-FreeRTOS` e incluye la opción `-Iconfig -IFreeRTOS-Kernel/include` en el comando `gcc`.

### 3. ❌ Error de enlace: `undefined reference to pthread_create` o `pthread_join`
* **Causa**: Se omitió la bandera de soporte para multihilo de Linux.
* **Solución**: Asegúrate de incluir `-pthread` dentro de los parámetros de compilación de `gcc`.

### 4. ❌ Error de compilación en el puerto POSIX: `No tal archivo o directorio` al compilar `port.c`
* **Causa**: En algunas versiones del repositorio de FreeRTOS, la ruta interna de `Posix` puede variar ligeramente.
* **Solución**: Ejecuta `find FreeRTOS-Kernel/portable -path "*Posix*" -name "port.c"` para ubicar la ruta exacta de tu sistema y reemplázala en el comando `gcc`.

### 5. 🔄 La terminal parece "bloqueada" o la salida se despliega indefinidamente
* **Causa**: El programa `cronograma` incluye un bucle infinito que redibuja la pantalla segundo a segundo mediante la tarea `vMonitor`.
* **Solución**: Para detener la ejecución y regresar al prompt de la terminal, presiona la combinación de teclas **`Ctrl + C`**.

---

## 📚 6. Referencias Bibliográficas

1. **[Documentación] RTOS Fundamentals - FreeRTOS™**: Documentación oficial de FreeRTOS.com sobre conceptos fundamentales de RTOS, prioridades de tareas, estados (`RUNNING`, `READY`, `BLOCKED`, `SUSPENDED`) y el funcionamiento del planificador preventivo (*Preemptive Scheduler*).
2. **[Documentacion] What is FreeRTOS? - FreeRTOS™**: Guía y visión general oficial de FreeRTOS.com sobre la arquitectura ligera del kernel, el minimalismo de memoria y la multitarea determinista en microcontroladores y emuladores.
3. **[Documentación] Mastering the FreeRTOS™ Real Time Kernel**: Manual de referencia oficial de Richard Barry sobre gestión de tareas, la tarea inactiva (*Idle Task Hook*), la configuración del archivo `FreeRTOSConfig.h` y el puerto POSIX para entornos Linux.
4. **[Manual] The FreeRTOS™ Reference Manual**: Especificación técnica y prototipos de funciones API como `xTaskCreate()`, `vTaskDelayUntil()`, `xQueueSend()` y `xQueueReceive()`.
