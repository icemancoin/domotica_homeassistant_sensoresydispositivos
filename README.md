# Domótica con Home Assistant: sensores y dispositivos

Colección de configuraciones **ESPHome** para dispositivos caseros que se integran en **Home Assistant**: dos cámaras ESP32-CAM y dos estaciones meteorológicas con ESP8266 (NodeMCU) y sensor DHT11.

## Dispositivos

| Dispositivo | Placa | Qué hace | Archivo |
|---|---|---|---|
| Cámara 1 | ESP32-CAM | Vídeo en streaming, snapshot, flash y reinicio | `cameras/espcam1.yaml` |
| Cámara 2 | ESP32-CAM | Igual que la cámara 1 | `cameras/espcam2.yaml` |
| Meteo oficina | NodeMCU V2 (ESP8266) | Temperatura y humedad con DHT11 | `weather/weather-dht-office.yaml` |
| Meteo salón | NodeMCU V2 (ESP8266) | DHT11 y pantalla OLED con temperatura, humedad y estado del WiFi | `weather/weather-station-oled.yaml` |

## Materiales

- 2 × ESP32-CAM (con PSRAM)
- 2 × NodeMCU V2 (ESP8266)
- 2 × sensor DHT11
- 1 × pantalla OLED SSD1306 128x64 (I2C, dirección 0x3C)
- Cables, protoboard y alimentación USB 5 V
- Un programador USB-serie (FTDI) para flashear la ESP32-CAM la primera vez
- Un equipo que ejecute Home Assistant (ver sección siguiente)

## Dispositivo que ejecuta Home Assistant

Estos dispositivos ESPHome necesitan un equipo encendido 24/7 en tu red local que ejecute Home Assistant. Ese equipo recibe los datos de los sensores, guarda el histórico y muestra las cámaras. Opciones:

| Opción | Cuándo elegirla | Notas |
|---|---|---|
| Home Assistant Green | Quieres algo listo para enchufar | Viene con el sistema preinstalado. No lleva radio Zigbee/Thread, hay que añadir un adaptador USB |
| Home Assistant Yellow | Quieres Zigbee integrado y NVMe | Radio Zigbee 3.0 integrada y ranura M.2 NVMe |
| Raspberry Pi (4 o 5) | Presupuesto ajustado, uso básico | La Pi 5 tiene PCIe para SSD NVMe. Mejor con SSD que con tarjeta SD |
| Mini PC x86-64 (Intel N100/N150 o un i5 antiguo) | Varias cámaras, Frigate, otros servicios | Necesita arrancar en UEFI y con Secure Boot desactivado |
| Máquina virtual (Proxmox, etc.) | Ya tienes un servidor | Permite ejecutar Home Assistant junto a otros servicios |
| NAS con Docker (p. ej. Synology) | Ya tienes un NAS potente | Home Assistant Container, sin tienda de complementos |

### Métodos de instalación

- **Home Assistant OS:** es el recomendado. Incluye Supervisor, complementos (add-ons) y copias de seguridad.
- **Home Assistant Container:** ejecuta el Core en Docker y lo gestionas tú. Sin Supervisor ni complementos, por lo que servicios como MQTT o ESPHome hay que desplegarlos como contenedores aparte.
- **Core y Supervised:** ya no están soportados, así que no los uses en una instalación nueva.

### Recursos orientativos

Para unos cientos de entidades y 30 días de histórico bastan 2 vCPU, 4 GB de RAM y 80 GB de SSD. Este proyecto tiene muy pocas entidades, así que cualquier opción de la tabla llega. Las cámaras con detección (Frigate) son lo que más exige, y para 5-10 cámaras conviene un mini PC con Intel QuickSync.

### Para este proyecto

- ESPHome se puede ejecutar como complemento de Home Assistant OS o instalarse con `pip` en otro PC.
- Las 4 placas deben estar en la misma red local que Home Assistant, para que la integración las detecte.
- Las cámaras ESP32-CAM generan tráfico continuo, así que usa red cableada o buen WiFi para el servidor.

## Conexiones

**ESP32-CAM**

| Función | Pin |
|---|---|
| Flash (LED) | GPIO4 |
| I2C SDA / SCL (cámara) | GPIO26 / GPIO27 |
| Reloj externo | GPIO0 |
| Pines de datos de cámara | GPIO5, 18, 19, 21, 36, 39, 34, 35 |
| VSYNC / HREF / PCLK | GPIO25 / GPIO23 / GPIO22 |
| Power down | GPIO32 |

Es la distribución estándar del modelo AI-Thinker.

**NodeMCU oficina:** DHT11 en el pin `D5`.

**NodeMCU salón:** DHT11 en `D1`, OLED en I2C con SDA en `D6` y SCL en `D5`.

## Instalación

1. Instala Home Assistant en uno de los equipos anteriores.
2. Instala [ESPHome](https://esphome.io) (el complemento de Home Assistant o `pip install esphome`).
3. Copia `secrets.example.yaml` a `secrets.yaml` y rellena tus datos reales:
   ```yaml
   wifi_ssid: "TU_WIFI"
   wifi_password: "TU_PASSWORD"
   ap_password: "CAMBIA_ESTA_CLAVE"
   ```
4. Compila y flashea el dispositivo que quieras:
   ```bash
   esphome run cameras/espcam1.yaml
   ```
   La primera vez se flashea por USB. Después, por OTA.
5. Home Assistant detectará el dispositivo en **Ajustes → Dispositivos y servicios** y podrás añadirlo.

## Entidades en Home Assistant

- **Cámaras:** imagen de cámara, luz de flash y botón de reinicio.
- **Meteo:** sensores de temperatura (°C) y humedad (%), con clase de dispositivo y `state_class: measurement`, listos para gráficas e histórico.

## Streams de las cámaras

Cada cámara expone un servidor web propio:

- Stream: `http://IP_DE_LA_CAMARA:8080`
- Snapshot: `http://IP_DE_LA_CAMARA:8081`

Configuración actual: 640x480, calidad JPEG 12, máximo 5 fps y 0,2 fps en reposo para reducir carga.

## Seguridad

- `secrets.yaml` está en `.gitignore` y nunca se sube.
- No expongas los puertos 8080 y 8081 ni Home Assistant directamente a internet. Para verlos desde fuera usa una VPN como Tailscale o un túnel con autenticación.
- Cambia la clave del AP de respaldo por una propia.

## Problemas habituales

- **DHT11 devuelve errores:** comprueba el cable de datos y activa `pullup: true` si tu módulo no lleva resistencia.
- **La ESP32-CAM no arranca al flashear:** conecta GPIO0 a GND durante la carga.
- **El OLED no muestra nada:** con `scan: true` mira en el log si detecta la dirección 0x3C.

## Licencia

MIT.
