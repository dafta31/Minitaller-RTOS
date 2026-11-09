# 🛠️ Minitaller Práctico: Sistemas Operativos de Tiempo Real con FreeRTOS

Minitaller para estudiantes de Ingeniería en Electrónica (ITCR). Enseña cómo un planificador por prioridades decide qué tarea tiene la CPU en cada momento, y cómo cambiar la prioridad, la duración y el periodo de las tareas cambia el resultado. Todo se hace en la terminal de Ubuntu, sin necesidad de hardware. La práctica de demostración termina con un Arduino Mega 2560.

---

## 1. ¿Qué incluye el repositorio?

El repositorio contiene dos materiales en Markdown:

- **Tutorial guiado (`Tutorial_Guiado.md`):** los estudiantes descargan el kernel de FreeRTOS, escriben dos archivos (`config/FreeRTOSConfig.h` y `src/main.c`), compilan con un solo comando de `gcc` y cambian prioridad, duración y periodo de las tareas para observar el cronograma.
- **Práctica de demostración (`Demostracion_practica.md`):** demostración de 15 a 20 minutos, a cargo del presentador. Ejecuta el demo BlinkyDemo del simulador POSIX de FreeRTOS y, al final, carga la tarea de parpadeo en un Arduino Mega 2560.

---

## 2. Estructura del repositorio

```
Minitaller-RTOS/
├── README.md                   Esta guía
├── Tutorial_Guiado.md          Tutorial guiado para estudiantes
└── Demostracion_practica.md    Demostración del presentador
```

El repositorio no contiene código compilado ni archivos del kernel. Los estudiantes obtienen el kernel desde GitHub durante el tutorial.

---

## 3. Requisitos

Ubuntu 22.04 o 24.04, nativo o en WSL2, con:

```bash
sudo apt update && sudo apt install -y git build-essential gcc nano
```

Para el tutorial guiado se necesita acceso a internet para clonar `FreeRTOS-Kernel`.

Para la demostración se necesita además:

- El repositorio oficial `FreeRTOS/FreeRTOS`, clonado con sus submódulos.
- Un Arduino Mega 2560, un LED, una resistencia de 330 Ω, una protoboard y jumpers.
- El Arduino IDE 2.3.10 (descargable desde la terminal, según el markdown de la práctica).

---

## 4. ¿Cómo usar los materiales?

1. Abran el tutorial guiado y sigan los pasos en orden. Cada paso que compila muestra el comando completo de `gcc`.

---

## 5. Referencias

- Documentación oficial de FreeRTOS: [freertos.org](https://www.freertos.org)
- Repositorio del kernel: [github.com/FreeRTOS/FreeRTOS-Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel)
- Repositorio completo con los demos: [github.com/FreeRTOS/FreeRTOS](https://github.com/FreeRTOS/FreeRTOS)
- Cuaderno de estudio del taller: [Sistemas operativos de tiempo real para sistemas embebidos: el caso FreeRTOS](https://gemini.google.com/notebook/14fd2157-5944-4d94-86c5-171664dad0ea)
