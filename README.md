# Control Arduino desde el navegador

Aplicación web simple para comunicarse con un Arduino, ESP32 u otro dispositivo compatible desde el navegador. Permite conectarse por USB o Bluetooth, enviar comandos de texto, visualizar datos recibidos y usar la cámara del equipo para mostrar una vista previa y capturar fotografías.

La aplicación está hecha únicamente con HTML, CSS y JavaScript. No utiliza un framework, un servidor backend ni un proceso de compilación.

## Funcionalidades

- Conexión con una placa por USB mediante Web Serial API.
- Conexión con un dispositivo BLE mediante Web Bluetooth API.
- Selección de velocidad de comunicación USB.
- Estado visual de la conexión.
- Envío de comandos de texto al dispositivo.
- Envío automático de salto de línea al terminar cada comando.
- Visualización de datos recibidos en una consola.
- Detección de desconexiones y errores.
- Activación de la cámara del equipo.
- Vista previa del video de la cámara.
- Captura de una fotografía en formato PNG.
- Detención de la cámara y liberación de sus recursos.
- Diseño sencillo y adaptable a pantallas pequeñas.

## Estructura del proyecto

```text
p1/
├── index.html
├── styles.css
└── README.md
```

### `index.html`

Contiene:

- La estructura visual de la aplicación.
- Los botones y controles de conexión.
- La consola de mensajes.
- Los elementos de video e imagen de la cámara.
- La carga del script externo de comunicación.
- Toda la lógica JavaScript de interacción.

### `styles.css`

Contiene únicamente los estilos visuales:

- Colores.
- Bordes.
- Espaciado.
- Tamaños.
- Estados de botones.
- Consola.
- Vista de cámara.
- Adaptación básica para celulares.

### `README.md`

Este archivo. Documenta la instalación, el funcionamiento y las limitaciones del proyecto.

## Dependencia externa

La aplicación carga este archivo desde jsDelivr:

```html
<script src="https://cdn.jsdelivr.net/gh/MauroBobyk/ArdComp@main/arduino-usb.js"></script>
```

Ese archivo define globalmente la clase `ArduinoUSB`, que luego se instancia desde `index.html`:

```javascript
const arduino = new ArduinoUSB({ lineEnding: '\n' });
```

La aplicación depende de que:

1. Haya conexión a Internet al abrir la página.
2. jsDelivr pueda acceder al repositorio de GitHub.
3. El archivo `arduino-usb.js` siga existiendo en la rama `main`.
4. El archivo mantenga disponible la clase global `ArduinoUSB`.

Si se quiere evitar la dependencia de Internet, se puede descargar una copia de `arduino-usb.js` y cargarla localmente, por ejemplo:

```html
<script src="arduino-usb.js"></script>
```

## Requisitos del navegador

### USB

La conexión USB utiliza la Web Serial API. Se necesita:

- Google Chrome o Microsoft Edge actualizado.
- Una página servida desde `localhost` o mediante HTTPS.
- Un dispositivo conectado por USB que exponga un puerto serie.
- Permiso del usuario para seleccionar el puerto.

Abrir el archivo directamente con `file://` puede impedir el uso de Web Serial.

### Bluetooth

La conexión Bluetooth utiliza Web Bluetooth API. Se necesita:

- Un navegador compatible, normalmente Chrome o Edge.
- Bluetooth activado en el equipo.
- Un dispositivo Bluetooth Low Energy, no Bluetooth clásico.
- Un servicio UART compatible.
- Permiso del usuario para seleccionar el dispositivo.
- Una página servida desde `localhost` o HTTPS.

La implementación del script externo utiliza por defecto el servicio UART Nordic UART Service:

```text
Servicio: 6e400001-b5a3-f393-e0a9-e50e24dcca9e
RX:       6e400002-b5a3-f393-e0a9-e50e24dcca9e
TX:       6e400003-b5a3-f393-e0a9-e50e24dcca9e
```

Estos UUID corresponden a configuraciones habituales de ESP32 BLE, nRF y algunos módulos UART BLE. El dispositivo debe utilizar esos UUID o la librería deberá configurarse para otros.

### Cámara

La cámara utiliza `navigator.mediaDevices.getUserMedia()` y requiere:

- Permiso del usuario.
- HTTPS o `localhost`.
- Una cámara disponible.
- Un navegador con soporte para MediaDevices.

## Cómo ejecutar el proyecto

No hace falta instalar Node.js ni dependencias de npm.

### Opción recomendada: servidor local con Python

Desde la carpeta del proyecto:

```bash
cd /home/mauro/ProyectosWeb/p1
python3 -m http.server 8000
```

Después abrir:

```text
http://localhost:8000
```

Para detener el servidor se puede presionar `Ctrl+C` en la terminal.

### Otras opciones

También se puede servir la carpeta con cualquier servidor web local. Por ejemplo, usando PHP:

```bash
php -S localhost:8000
```

La dirección debe abrirse en un navegador compatible, no dentro de una vista previa que no permita Web Serial, Web Bluetooth o cámara.

## Interfaz

La aplicación contiene un único panel principal.

### Estado de conexión

El elemento con `id="status"` muestra el estado actual.

Estado inicial:

```text
Desconectado
```

Cuando se conecta correctamente, el texto cambia a:

```text
Conectado por USB
```

o:

```text
Conectado por Bluetooth
```

El punto de estado es rojo cuando no hay conexión y verde cuando existe una conexión activa. La clase CSS `connected` se agrega o elimina dinámicamente mediante JavaScript.

### Selector de baudios

El elemento `select` con `id="baud-rate"` ofrece estas velocidades:

- `9600` para Arduino.
- `115200` para ESP32, seleccionado por defecto.
- `19200`.
- `38400`.

El valor elegido se transforma en número y se envía a `connectUSB()` al conectar por USB.

El baud rate solo se usa para USB. Bluetooth no utiliza esta selección porque BLE trabaja con su propio protocolo de comunicación.

### Botón Conectar por USB

El botón con `id="usb-button"` ejecuta:

```javascript
arduino.connectUSB({
  baudRate: Number(document.getElementById('baud-rate').value)
});
```

El navegador abre el selector de puertos serie. El usuario debe seleccionar la placa o el puerto correspondiente.

Si la conexión falla, el mensaje del error se agrega a la consola de la interfaz.

Errores habituales:

- El navegador no soporta Web Serial.
- No se seleccionó ningún puerto.
- El puerto está ocupado por otro programa.
- El dispositivo fue desconectado.
- La velocidad seleccionada no coincide con el programa de la placa.

### Botón Conectar por Bluetooth

El botón con `id="bluetooth-button"` ejecuta:

```javascript
arduino.connectBluetooth();
```

El navegador abre el selector de dispositivos BLE. Se debe seleccionar un módulo compatible con el servicio UART esperado.

Errores habituales:

- El navegador no soporta Web Bluetooth.
- El Bluetooth del equipo está apagado.
- El dispositivo no es BLE.
- El dispositivo no expone el servicio o las características UART esperadas.
- Se canceló el selector de dispositivos.

### Botón Desconectar

El botón con `id="disconnect-button"` comienza deshabilitado. Se activa después de una conexión exitosa.

Ejecuta:

```javascript
arduino.disconnect();
```

La desconexión:

- Cierra el puerto USB o la conexión BLE.
- Detiene la lectura del puerto.
- Libera referencias internas.
- Actualiza el texto de estado.
- Registra el mensaje `Desconectado.` en la consola.

### Campo de comando

El campo con `id="command"` permite escribir un texto para enviar a la placa.

El botón `Enviar` permanece deshabilitado hasta que haya una conexión activa.

Presionar `Enter` dentro del campo tiene el mismo efecto que presionar `Enviar`.

Los comandos vacíos se ignoran.

### Botón Enviar

El botón envía el texto usando:

```javascript
arduino.sendLine(command);
```

`sendLine()` agrega el final de línea configurado, que en este proyecto es `\n`.

Por ejemplo, si se escribe:

```text
LED:ON
```

la placa recibe los bytes equivalentes a:

```text
LED:ON\n
```

Después de enviar correctamente:

- El comando aparece en la consola con la flecha `→`.
- El campo de entrada se limpia.

## Consola

La consola es el elemento con `id="console"`.

Se utiliza para mostrar:

- El estado inicial de las APIs disponibles.
- Conexiones exitosas.
- Datos recibidos.
- Comandos enviados.
- Errores.
- Activación y detención de la cámara.
- Capturas realizadas.

Ejemplos de mensajes:

```text
Listo para conectar.
✓ Conectado por usb
→ LED:ON
← Temperatura: 24.3
Desconectado.
```

La consola se desplaza automáticamente hacia abajo cada vez que se agrega un mensaje.

La función responsable es:

```javascript
function log(message) {
  if (consoleOutput.textContent === 'Cargando arduino-usb.js...') {
    consoleOutput.textContent = '';
  }

  consoleOutput.textContent += `${message}\n`;
  consoleOutput.scrollTop = consoleOutput.scrollHeight;
}
```

## Datos recibidos

El objeto `ArduinoUSB` emite un evento `data` cada vez que recibe datos.

La aplicación lo escucha así:

```javascript
arduino.addEventListener('data', (event) => {
  log(`← ${event.detail}`);
});
```

El contenido recibido se muestra directamente en la consola. La aplicación no convierte el dato en JSON ni aplica un protocolo propio.

La placa puede enviar texto con cualquier formato que su programa soporte, por ejemplo:

```text
OK
TEMP:25
LED:ON
DISTANCIA:120
```

## Eventos utilizados

La aplicación utiliza cuatro eventos del objeto `ArduinoUSB`.

### `connect`

Se emite después de una conexión USB o Bluetooth exitosa.

La aplicación:

- Activa el estado visual de conexión.
- Identifica el transporte usado.
- Habilita `Desconectar`.
- Habilita `Enviar`.
- Agrega un mensaje a la consola.

### `data`

Se emite cuando llegan datos desde la placa. La aplicación los agrega a la consola con una flecha hacia la izquierda.

### `disconnect`

Se emite cuando se llama a `disconnect()` o cuando se detecta que la conexión terminó.

La aplicación:

- Vuelve el estado a `Desconectado`.
- Deshabilita `Desconectar`.
- Deshabilita `Enviar`.
- Agrega un mensaje a la consola.

### `error`

Se emite cuando el script externo detecta un error de comunicación. El mensaje se agrega a la consola con el prefijo `⚠`.

## Cámara

La cámara no depende de Arduino. Es una funcionalidad independiente del navegador.

### Activar cámara

El botón `Cámara` solicita video sin audio:

```javascript
navigator.mediaDevices.getUserMedia({
  video: true,
  audio: false
});
```

Cuando el usuario acepta:

- El stream se asigna al elemento `<video>`.
- Se muestra la vista previa.
- Se habilita `Capturar foto`.
- Se habilita `Detener cámara`.
- Se registra `✓ Cámara activada.`.

### Capturar foto

El botón `Capturar foto` utiliza un elemento `canvas` creado en memoria.

El flujo es:

1. Comprueba que el video tenga dimensiones válidas.
2. Crea un canvas del mismo tamaño que el video.
3. Dibuja el fotograma actual del video.
4. Convierte el canvas a PNG mediante `toDataURL('image/png')`.
5. Coloca el resultado en el elemento `<img id="captured-photo">`.
6. Muestra la imagen capturada.
7. Registra `✓ Foto capturada.`.

La fotografía no se guarda automáticamente en el disco ni se sube a ningún servidor. Solo queda disponible en la página mientras esta permanezca abierta.

### Detener cámara

El botón `Detener cámara` detiene todas las pistas del stream:

```javascript
cameraStream.getTracks().forEach((track) => track.stop());
```

Luego:

- El video deja de mostrar la cámara.
- Se eliminan las referencias al stream.
- Se oculta la vista previa.
- Se deshabilitan los botones de captura y detención.
- Se registra `Cámara detenida.`.

## Programa mínimo para Arduino

La aplicación envía texto terminado en salto de línea. Un programa Arduino mínimo podría leer líneas así:

```cpp
void setup() {
  Serial.begin(115200);
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  if (Serial.available()) {
    String comando = Serial.readStringUntil('\n');
    comando.trim();

    if (comando == "LED:ON") {
      digitalWrite(LED_BUILTIN, HIGH);
      Serial.println("LED encendido");
    }

    if (comando == "LED:OFF") {
      digitalWrite(LED_BUILTIN, LOW);
      Serial.println("LED apagado");
    }
  }
}
```

La velocidad de `Serial.begin()` debe coincidir con la opción seleccionada en la página. En el ejemplo se usa `115200`.

Para un Arduino configurado a `9600`, se debe usar:

```cpp
Serial.begin(9600);
```

y seleccionar `9600 — Arduino` en la interfaz.

## Organización del JavaScript

El JavaScript está actualmente dentro de `index.html`, después de cargar `arduino-usb.js`.

### Referencias a elementos

Al principio se guardan referencias a los elementos del DOM:

```javascript
const status = document.getElementById('status');
const statusText = document.getElementById('status-text');
const consoleOutput = document.getElementById('console');
```

Esto evita buscar continuamente los mismos elementos en el documento.

### Variable `cameraStream`

La variable:

```javascript
let cameraStream = null;
```

guarda el stream de la cámara mientras está activo. Es necesaria para poder detener sus pistas posteriormente.

### Función `setConnected`

Esta función centraliza la actualización del estado de conexión:

- Cambia la clase `connected`.
- Cambia el texto de estado.
- Habilita o deshabilita los botones relacionados.

### Manejo de promesas

Las conexiones, el envío y la cámara son operaciones asíncronas. Por eso sus handlers utilizan `async`, `await` y `try/catch`.

Los errores no interrumpen toda la página: se capturan y se muestran en la consola de la interfaz.

## CSS y diseño

El proyecto usa un diseño deliberadamente discreto.

Características principales:

- Fondo gris muy claro.
- Panel blanco con borde gris.
- Bordes redondeados leves.
- Botones grises sin efectos intensos.
- Indicador rojo o verde para el estado.
- Consola con tipografía monoespaciada.
- Sin dependencias visuales externas.
- Sin iconos, fuentes ni frameworks adicionales.

La regla `@media (max-width: 560px)` permite que los botones se acomoden en varias líneas en pantallas pequeñas.

## Seguridad y permisos

El navegador controla directamente los permisos de:

- Puerto serie USB.
- Dispositivos Bluetooth.
- Cámara.

La página no puede acceder silenciosamente a esos recursos. Cada permiso debe ser aprobado por el usuario mediante el selector o el diálogo del navegador.

La aplicación tampoco contiene un backend. Los datos se transmiten directamente desde el navegador al dispositivo seleccionado.

## Solución de problemas

### Aparece `Web Serial no soportado`

Usar Chrome o Edge actualizado y abrir la aplicación desde `localhost` o HTTPS.

### No aparece el puerto USB

Comprobar:

- Que el cable USB transmita datos y no sea únicamente de carga.
- Que la placa esté conectada.
- Que otro programa no esté usando el puerto.
- Que el controlador USB/serie esté instalado.
- Que se haya presionado el botón de conexión desde la página.

### La placa se conecta, pero no responde

Comprobar:

- El baud rate.
- El salto de línea esperado por el programa Arduino.
- Que el programa lea con `Serial.readStringUntil('\n')` o una lógica equivalente.
- Que el comando coincida exactamente con el texto esperado.

### Aparece un error de Bluetooth

Comprobar que:

- El dispositivo sea BLE.
- El servicio UART coincida con los UUID configurados.
- El módulo esté encendido y dentro del alcance.
- El navegador tenga Bluetooth disponible.

### La cámara no funciona

Comprobar que:

- Se haya aceptado el permiso.
- La página se abra por `localhost` o HTTPS.
- Otra aplicación no esté usando la cámara.
- El navegador tenga acceso al dispositivo de cámara.

### El script remoto no carga

Comprobar:

- La conexión a Internet.
- La URL de jsDelivr.
- Que el repositorio `MauroBobyk/ArdComp` sea accesible.
- Que exista `arduino-usb.js` en la rama `main`.
- La consola de desarrollador del navegador para ver el error de red.

## Limitaciones actuales

- No hay guardado permanente de fotografías.
- No hay historial separado de mensajes.
- No hay selección manual de cámara.
- No hay configuración visual para UUID Bluetooth.
- No hay reconexión automática.
- No hay autenticación de usuarios.
- No hay servidor ni base de datos.
- El JavaScript todavía está dentro de `index.html`.
- La aplicación depende de un script externo alojado en GitHub/jsDelivr.
- Web Bluetooth no funciona en todos los navegadores ni con todos los módulos.

## Posibles mejoras futuras

Algunas mejoras posibles son:

- Mover el JavaScript propio a `app.js`.
- Guardar fotografías mediante descarga local.
- Agregar un botón para limpiar la consola.
- Agregar un selector de cámaras.
- Permitir configurar el fin de línea.
- Permitir configurar los UUID Bluetooth desde la interfaz.
- Agregar comandos predefinidos.
- Agregar timestamps a los mensajes.
- Agregar reconexión manual guiada.
- Incorporar validaciones más específicas para cada tipo de placa.

## Resumen del flujo completo

```text
1. El navegador carga index.html.
2. index.html carga styles.css.
3. index.html carga arduino-usb.js desde jsDelivr.
4. Se crea una instancia de ArduinoUSB.
5. Se comprueba si USB y Bluetooth están disponibles.
6. El usuario elige un baud rate.
7. El usuario conecta por USB o Bluetooth.
8. El navegador solicita el permiso correspondiente.
9. La aplicación actualiza el estado visual.
10. El usuario puede enviar comandos terminados en salto de línea.
11. Los datos recibidos aparecen en la consola.
12. El usuario puede activar y capturar la cámara de forma independiente.
13. Al desconectar, la aplicación actualiza el estado y libera la conexión.
```

## Estado del proyecto

Proyecto funcional de frontend estático para pruebas y control básico de dispositivos Arduino/ESP32 desde el navegador.
