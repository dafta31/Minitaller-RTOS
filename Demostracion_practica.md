### Práctica guiada: ejecutar el demo Posix_GCC de FreeRTOS (BLINKY_DEMO)
**Duración:**  15 a 20 minutos  **¿Qué se va a realizar?:**  descargar el repositorio oficial de FreeRTOS con sus submódulos, compilar el demo del simulador POSIX y ejecutar el demo  **BLINKY_DEMO** . Finalmente, se carga un script de FreeRTOS en el arduino que parpadea un LED.

--------------------------------------------------------------------------------

#### ¿Cómo usar nano?
*  Ctrl+W busca texto. Escriban el texto en la línea Buscar: de abajo y presionen Enter.
*  Las flechas mueven el cursor. Backspace borra el carácter a la izquierda.
*  Ctrl+O y luego Enter guardan el archivo.
*  Ctrl+X sale de nano.

--------------------------------------------------------------------------------

#### Paso 1: Instalar las herramientas
```bash
sudo apt-get update
sudo apt-get install -y build-essential git nano

```
**Verificación:**  los paquetes se instalan sin errores.

--------------------------------------------------------------------------------

#### Paso 2: Clonar el repositorio oficial con submódulos
El repositorio pesa entre 150 y 300 MB, así que puede tardar varios minutos.
```bash
cd ~
git clone https://github.com/FreeRTOS/FreeRTOS.git --recurse-submodules

```
**Verificación:**  la carpeta de la demo existe:
```bash
ls ~/FreeRTOS/FreeRTOS/Demo/Posix_GCC

```
Deben aparecer los archivos fuente y el Makefile.

--------------------------------------------------------------------------------

#### Paso 3: Entrar a la carpeta de la demo
```bash
cd ~/FreeRTOS/FreeRTOS/Demo/Posix_GCC
ls

```
**Verificación:**  aparecen Makefile y main.c.
Guarden una copia del main.c original, para poder volver a él al final de la práctica:
```bash
cp main.c ~/main_original.c

```

--------------------------------------------------------------------------------

#### Paso 4: Revisar la selección del demo en main.c
Antes de editar, revisen qué valor tiene la configuración por defecto:
```bash
sed -n '74,92p' main.c

```
Deben ver estas líneas:
```c
#endif

#define    BLINKY_DEMO    0
#define    FULL_DEMO      1

#ifdef BUILD_DIR
    #define BUILD         BUILD_DIR
#else
    #define BUILD         "./"
#endif

/* Demo type is passed as an argument */
#ifdef USER_DEMO
    #define     mainSELECTED_APPLICATION    USER_DEMO
#else /* Default Setting */
    #define    mainSELECTED_APPLICATION     FULL_DEMO
#endif

/* This demo uses heap_3.c (the libc provided malloc() and free()). */

```
BLINKY_DEMO vale 0 y FULL_DEMO vale 1. La línea que decide el demo es la del bloque #else.
En la versión que revisamos, el bloque #else dice FULL_DEMO. Para ejecutar BlinkyDemo,  **hagan el Paso 5**  y cambien el valor a BLINKY_DEMO.
**Verificación:**  anoten el valor que aparece en el bloque #else.

--------------------------------------------------------------------------------

#### Paso 5: Dejar BLINKY_DEMO por defecto
Hagan este paso si el bloque #else dice FULL_DEMO, como en la versión que revisamos.
```bash
nano main.c

```
Dentro de nano:
1. Presionen Ctrl+W, escriban Default Setting y presionen Enter. El cursor queda en la línea #else /* Default Setting */. Presionen la flecha abajo una vez para bajar a la línea #define    mainSELECTED_APPLICATION     FULL_DEMO.
2. Cambien FULL_DEMO por BLINKY_DEMO.
La línea debe quedar así:
```c
    #define    mainSELECTED_APPLICATION     BLINKY_DEMO

```
3. Presionen Ctrl+O, Enter y Ctrl+X.
**Verificación:**
```bash
sed -n '74,92p' main.c

```
La línea del bloque #else debe mostrar BLINKY_DEMO.

--------------------------------------------------------------------------------

#### Paso 6: Compilar el demo BLINKY_DEMO
```bash
cd ~/FreeRTOS/FreeRTOS/Demo/Posix_GCC
make clean
make

```
**Verificación:**  la compilación termina sin errores y se crea la carpeta build. Revisen que el ejecutable exista:
```bash
ls build

```
Si make falla, copien el mensaje de error completo antes de seguir.

--------------------------------------------------------------------------------

#### Paso 7: Ejecutar BLINKY_DEMO
```bash
cd build
./posix_demo

```
**Verificación:**  aparecen estas líneas al inicio:
```
Trace started.
Starting echo blinky demo

```
Después aparecen mensajes continuos Message received from task y Message received from software timer.
Para detenerlo presionen Ctrl+C.
Después de detenerlo, regresen a la carpeta del demo:
```bash
cd ~/FreeRTOS/FreeRTOS/Demo/Posix_GCC

```

--------------------------------------------------------------------------------

#### Paso 8: Restaurar el archivo original
```bash
cd ~/FreeRTOS/FreeRTOS/Demo/Posix_GCC
cp ~/main_original.c main.c
make clean
make

```

--------------------------------------------------------------------------------

#### Paso 9: Demostración en Arduino Mega 2560
**Materiales:**  Arduino Mega 2560, cable USB, un LED, una resistencia de 220 Ω, protoboard y dos cables.
##### 9.1 Descargar e instalar Arduino IDE desde la terminal
Descarguen el AppImage del Arduino IDE 2.3.10 a la carpeta Downloads:
```bash
cd ~/Downloads
wget https://downloads.arduino.cc/arduino-ide/arduino-ide_2.3.10_Linux_64bit.AppImage

```
**Si wget no está instalado**  (aparece wget: command not found), tienen dos opciones:
*  Instalarlo y repetir la descarga:
```bash
sudo apt-get install -y wget
wget https://downloads.arduino.cc/arduino-ide/arduino-ide_2.3.10_Linux_64bit.AppImage

```
*  O usar curl, que normalmente ya viene instalado en Ubuntu:
```bash
curl -L -O https://downloads.arduino.cc/arduino-ide/arduino-ide_2.3.10_Linux_64bit.AppImage

```
**Si la descarga responde con error (por ejemplo 404),**  la dirección cambió. Descarguen el AppImage Linux 64 bits desde arduino.cc/en/software y guárdenlo en ~/Downloads.
Verifiquen que el archivo existe:
```bash
ls -lh ~/Downloads/arduino-ide*

```
Den permiso de ejecución al archivo:
```bash
chmod +x arduino-ide_2.3.10_Linux_64bit.AppImage

```
Abran el IDE con --no-sandbox:
```bash
./arduino-ide_2.3.10_Linux_64bit.AppImage --no-sandbox

```
La opción --no-sandbox desactiva el aislamiento de seguridad de Electron. Es necesaria en algunas versiones de Ubuntu para que el IDE abra desde un AppImage. Úsenla solo con este archivo descargado desde el sitio oficial de Arduino.
**Verificación:**  se abre la ventana del Arduino IDE sin errores.
##### 9.2 Instalar la biblioteca de FreeRTOS para Arduino
En el IDE: Herramientas → Administrar bibliotecas. Busquen FreeRTOS, elijan la biblioteca que incluye Arduino_FreeRTOS.h y presionen Instalar.
**Verificación:**  en una terminal:
```bash
find ~/Arduino -name "Arduino_FreeRTOS.h" 2>/dev/null

```
Debe aparecer una ruta. Si no aparece, instalen la biblioteca desde el Administrador de bibliotecas y repitan la búsqueda.
##### 9.3 Conectar el LED
Conecten el pin 22 del Arduino a una resistencia de 220 Ω. La otra punta de la resistencia va a la pata larga del LED (ánodo). La pata corta (cátodo) va a GND.
**Verificación:**  la conexión está en la protoboard y el LED está en el sentido correcto (pata larga hacia la resistencia).
##### 9.4 Seleccionar placa y puerto
Conecten el Arduino por USB y en el IDE elijan:
*  Herramientas → Placa → Arduino Mega or Mega 2560
*  Herramientas → Puerto → el puerto que aparezca, por ejemplo /dev/ttyACM0
Si no aparece ningún puerto, ejecuten en la terminal:
```bash
ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null

```
Si aparece un puerto pero el IDE no puede subir el código, agreguen su usuario al grupo dialout, cierren sesión y vuelvan a entrar:
```bash
sudo usermod -aG dialout $USER

```
##### 9.5 Escribir el código
En el IDE, Archivo → Nuevo, y peguen este código completo:
```cpp
#include <Arduino_FreeRTOS.h>

// Define the LED pin (you can change this to 2, 22, 24, etc.)
const int ledPin = 22;

// Declare the task function prototype
void TaskBlink(void *pvParameters);

void setup() {
  // Set the LED digital pin as an output
  pinMode(ledPin, OUTPUT);

  // Create the FreeRTOS task
  xTaskCreate(
    TaskBlink,    // Task function
    "BlinkTask",  // Descriptive task name (for debugging)
    256,          // Stack size in words
    NULL,         // Task input parameter
    1,            // Priority (1 is low priority)
    NULL          // Task handle
  );

  // Note: On AVR-based Arduino boards like the Mega, the feilipu
  // FreeRTOS library automatically starts the scheduler when setup() finishes.
  // Therefore, loop() remains empty.
}

void loop() {
  // The FreeRTOS scheduler handles all task execution.
  // Leaving loop() empty prevents blocking the scheduler.
}

/*--------------------------------------------------*/
/* Task Implementation                              */
/*--------------------------------------------------*/
void TaskBlink(void *pvParameters) {
  (void) pvParameters; // Unused parameter

  for (;;) {
    digitalWrite(ledPin, HIGH);           // Turn the LED on
    vTaskDelay(500 / portTICK_PERIOD_MS); // Block task for 500ms (non-blocking for OS)

    digitalWrite(ledPin, LOW);            // Turn the LED off
    vTaskDelay(500 / portTICK_PERIOD_MS); // Block task for 500ms
  }
}

```
Guarden el archivo con Ctrl+S. El IDE les pedirá un nombre: escriban BlinkTask.
##### 9.6 Verificar y subir el código
1. Presionen el botón ✓ (Verificar). Esperen el mensaje de compilación sin errores.
2. Presionen el botón → (Subir). Esperen el mensaje de carga finalizada.
**Verificación:**  en la parte inferior del IDE aparece Subida completa o un mensaje equivalente, sin errores.
##### 9.7 Observar el parpadeo
El LED se enciende 500 ms y se apaga 500 ms, de forma continua. Es la tarea BlinkTask corriendo bajo el planificador de FreeRTOS.
**Preguntas para la clase:**
1. ¿Qué pasaría con el parpadeo si vTaskDelay tuviera 1000 en lugar de 500?
2. ¿Por qué loop() queda vacío?

--------------------------------------------------------------------------------

#### Preguntas de cierre
1. ¿Qué constante de main.c decide qué demo se ejecuta?
2. ¿Por qué hay que ejecutar make clean antes de make cada vez que cambian la selección del demo?
3. ¿Qué mensajes aparecen al inicio del demo y qué significa Starting echo blinky demo?

--------------------------------------------------------------------------------

#### Recurso de estudio
Cuaderno del taller: [Sistemas operativos de tiempo real para sistemas embebidos: el caso FreeRTOS](https://gemini.google.com/notebook/14fd2157-5944-4d94-86c5-171664dad0ea)
