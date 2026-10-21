# Cafetera automatizada con PIC32

**Control de una cafetera mediante un microcontrolador, sensores, actuadores y una aplicación móvil.**

Proyecto de **Isabel Alonso Casas** y **Bibiana Dorado Mateos**, desarrollado para la asignatura de **Microprocesadores** en la **Universidad Pontificia Comillas — ICAI**.

El sistema permite elegir un café corto, normal o largo, con o sin azúcar, y coordina las etapas de calentamiento, dosificación, mezcla y servicio mediante una máquina de estados programada en C.

**Tecnologías:** C · PIC32MX230F064D · MPLAB X · XC32 · UART · Bluetooth Low Energy · ADC · PWM · TFT · MIT App Inventor

## Vídeo de demostración

### [Ver el vídeo de la cafetera](https://upcomillas-my.sharepoint.com/:v:/g/personal/202301114_alu_comillas_edu/IQCw-3xlDj8JQKc9DCW5or0uARFAcWHtTdljE3raWDFIOhw)

El vídeo está alojado en SharePoint y se abre desde el enlace. Su reproducción depende de los permisos de acceso configurados por su propietaria.

[Leer la memoria del proyecto](docs/informe-final.pdf) · [Ver el código principal](firmware/DemoTFT.X/main.c) · [Descargar el proyecto de la app](app/AppCafetera.aia)

## Diseño de la cafetera

![Vistas del diseño de la cafetera y esquema de la aplicación móvil](docs/img/diseno-cafetera.jpeg)

*Vistas del diseño extraídas de la memoria original.*

## Funcionalidades

- Selección de **café corto, normal o largo**, con o sin azúcar, desde la app.
- Recepción de pedidos mediante **Bluetooth Low Energy y UART**.
- Inicio alternativo desde el pulsador de la placa con un café normal con azúcar.
- Comprobación de temperatura mediante un **termistor NTC y el ADC** del microcontrolador.
- Control de electroválvula, calefactor, motor de mezcla y bomba mediante relés.
- Dosificación de café y azúcar mediante **dos servomotores controlados por PWM**.
- Presentación del estado del proceso en una **pantalla TFT** y mediante los LED de la placa.
- Detección del vaso con un sensor infrarrojo: pausa de la bomba cuando se retira durante el servicio y reanudación al volver a colocarlo.
- Espera a que se retire el café terminado antes de admitir un nuevo ciclo.

La memoria establece un umbral nominal de **35 °C para la demostración**, con el objetivo de reducir el tiempo de calentamiento del prototipo.

## Funcionamiento

1. **Pedido:** la aplicación envía la selección de café y azúcar; también puede utilizarse el pulsador de la placa.
2. **Comprobación y calentamiento:** el sistema espera a que el sensor alcance el umbral de temperatura previsto.
3. **Dosificación:** la electroválvula introduce agua y los servomotores dosifican el café y, si se solicita, el azúcar.
4. **Mezcla:** el motor permanece activo durante el tiempo programado.
5. **Servicio:** la bomba lleva la mezcla al vaso; si el vaso se retira, el sistema pasa al estado de pausa.
6. **Fin:** la pantalla indica que el café está listo y el sistema espera a que se retire el vaso para reiniciar el ciclo.

![Máquina de estados del proyecto](docs/img/maquina-estados.png)

*Diagrama original de la memoria. Los nombres y las transiciones implementadas pueden consultarse en `main.c`.*

## Organización del software

El firmware se organiza en módulos para separar el control general, los periféricos y la comunicación:

| Archivo o módulo | Función |
| --- | --- |
| `main.c` | Máquina de estados y coordinación del proceso |
| `config.c` / `config.h` | Asignación e inicialización de pines |
| `Pic32Ini.c` / `Pic32Ini.h` | Configuración del reloj del microcontrolador |
| `UART1.c` / `UART1.h` | Comunicación serie y recepción de órdenes |
| `BitWhacker.c` / `BitWhacker.h` | Interpretación de los comandos de café y azúcar |
| `sensores.c` / `sensores.h` | Lectura del sensor infrarrojo y del ADC de temperatura |
| `actuadores.c` / `actuadores.h` | Encendido y apagado de los actuadores mediante relés |
| `servo.c` / `servo.h` | Control de los dos servomotores |
| `timers.c` / `timers.h` | Base temporal para el PWM y los tiempos del proceso |
| `Pantalla.c` / `Pantalla.h` | Interfaces y mensajes de la pantalla TFT |
| `TftDriver/` | Controlador de la pantalla, fuentes, fichas técnicas y licencias originales |

<details>
<summary>Ver el esquema original de la arquitectura del software</summary>

![Arquitectura del software recogida en la memoria](docs/img/arquitectura-software.png)

</details>

Se conservan también archivos de demostración y pruebas del proyecto original. El programa principal seleccionado en la configuración de MPLAB es `main.c`.

## Aplicación y comunicación

La app se incluye en **`app/AppCafetera.aia`**, un archivo de proyecto editable de MIT App Inventor. Incluye la interfaz, los bloques de programación y la extensión Bluetooth Low Energy utilizada.

La aplicación permite buscar dispositivos, conectarse, seleccionar el tipo de café y el azúcar, enviar el pedido y consultar una consola de comunicación.

El firmware configura UART a **9600 baudios, 8 bits de datos, sin paridad y un bit de parada**. Las órdenes siguen el formato `tipo,azucar`, terminado en salto de línea (`\n`):

| Parte del comando | Valores |
| --- | --- |
| Tipo de café | `C` = corto · `N` = normal · `L` = largo |
| Azúcar | `Y` = con azúcar · `N` = sin azúcar |

Por ejemplo, `N,Y` solicita un café normal con azúcar y `C,N` un café corto sin azúcar. La app incluye un control **ENDL** para el final de línea. El programa responde por UART con mensajes como `Pedido recibido`.

Los bloques de la app contienen un filtro de nombre `BT05`, el servicio BLE `FFE0` y la característica `FFE1`. Deben corresponder con el módulo Bluetooth del montaje.

## Contenido del repositorio

| Carpeta o archivo | Contenido |
| --- | --- |
| [firmware/DemoTFT.X](firmware/DemoTFT.X) | Proyecto MPLAB X, código C, cabeceras y controlador TFT |
| [app/AppCafetera.aia](app/AppCafetera.aia) | Proyecto original de la aplicación móvil |
| [docs/informe-final.pdf](docs/informe-final.pdf) | Memoria con el diseño, los componentes y el funcionamiento |
| [docs/img](docs/img) | Esquemas extraídos de la memoria para este README |
| `.gitignore` | Exclusión de archivos temporales y resultados de compilación |

## Cómo abrir el proyecto

### Firmware del PIC32

El proyecto original registra este entorno:

| Componente | Versión o dispositivo |
| --- | --- |
| IDE | MPLAB X IDE 6.20 |
| Compilador | MPLAB XC32 4.60 |
| Microcontrolador | PIC32MX230F064D |
| Paquete de dispositivos | Microchip PIC32MX_DFP 1.5.259 |

1. Descarga el repositorio desde **Code → Download ZIP** y descomprímelo.
2. Abre **MPLAB X IDE**.
3. Selecciona **File → Open Project** y abre la carpeta **`firmware/DemoTFT.X`**.
4. En las propiedades del proyecto, comprueba el dispositivo, el compilador XC32 instalado y el paquete de dispositivos. Ajusta las rutas a tu instalación si el IDE lo solicita.
5. Compila el proyecto desde MPLAB X. Para ejecutarlo en el montaje es necesario disponer de la placa, los periféricos y un programador o depurador compatible.

El programa está destinado al microcontrolador y al montaje físico del proyecto. La consulta del código y de la memoria puede hacerse directamente desde GitHub.

### Aplicación móvil

1. Accede a [MIT App Inventor](https://ai2.appinventor.mit.edu/).
2. Selecciona **Projects → Import project (.aia) from my computer**.
3. Importa **`app/AppCafetera.aia`**.
4. Revisa la interfaz en **Designer** y la lógica en **Blocks**.

El archivo `.aia` contiene el proyecto editable. Para probar la comunicación se necesita un dispositivo móvil y el módulo BLE del montaje compatibles con la extensión incluida.

## Preparación de esta copia

Se conservan sin modificaciones los archivos C y sus cabeceras, el proyecto de la app y la memoria. Se ha simplificado la carpeta anidada del ZIP original para dejar un único proyecto en `firmware/DemoTFT.X`.

Se han omitido resultados de compilación, cachés, registros, preferencias locales y documentación HTML generada. En `nbproject/configurations.xml` se han retirado únicamente cuatro referencias antiguas a cabeceras de MCC que no estaban presentes en el ZIP original y que el código conservado no incluye.

La preparación del repositorio no incluye una nueva compilación con XC32 ni una prueba sobre la placa física.

## Autoras y créditos

**Isabel Alonso Casas · Bibiana Dorado Mateos**  
Microprocesadores · Universidad Pontificia Comillas — ICAI

Se mantienen los créditos del código de apoyo original. El controlador TFT identifica a **José Daniel Muñoz Frías** y el módulo SPI incluye también a **Carlos Rodríguez-Morcillo**. Sus avisos y documentos de licencia se conservan en [TftDriver/License](firmware/DemoTFT.X/TftDriver/License).

Referencias de las herramientas: [MPLAB X IDE](https://www.microchip.com/en-us/tools-resources/develop/mplab-x-ide), [apertura de proyectos en MPLAB X](https://developerhelp.microchip.com/xwiki/bin/view/software-tools/ides/x/projects/operations/) y [MIT App Inventor](https://appinventor.mit.edu/).
