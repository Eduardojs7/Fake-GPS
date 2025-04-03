# Proyecto de Simulación de Fake GPS en Wokwi con Flespi y Grafana
## Ssanchez Salinas Eduardo Josue

## Descripción
Este proyecto simula el envío de datos GPS utilizando un microcontrolador de **Wokwi**, conectado a un broker MQTT en **Flespi**. Los datos simulados (latitud, longitud y timestamp) se publican en tiempo real y pueden ser visualizados en **Grafana** utilizando el plugin **GeoMap** para visualizar la ubicación.

## Requisitos
- **Wokwi** para la simulación del microcontrolador (ESP32 o similar).
- **Flespi** para gestionar los mensajes MQTT.
- **Grafana** para visualizar los datos en un mapa usando el plugin **GeoMap**.
- Conexión a **WiFi**.

## Estructura del Proyecto
Este repositorio contiene el código en **MicroPython** para simular los datos GPS y enviarlos a **Flespi** a través de MQTT.

### Código del Microcontrolador en Wokwi
El siguiente código simula el envío de datos GPS (latitud, longitud, y timestamp) desde el dispositivo simulado en **Wokwi** hacia **Flespi**:

```python
import network
import utime
import ujson
from umqtt.simple import MQTTClient
from machine import UART

# Configuración WiFi y MQTT
WIFI_SSID = "Wokwi-GUEST"
WIFI_PASS = ""
MQTT_BROKER = "mqtt.flespi.io"
MQTT_PORT = 1883
MQTT_TOPIC = "ebike/gps"
MQTT_TOKEN = "vvYG5uL0ytWGDqWL9bwSnj9ggoeAmDeh0MADgRjCjgZtHD2IjJKGW1hh9kHetLSb"

# Simulación de ruta GPS
ROUTE = [
    (19.440472, -99.204417),
    (19.440694, -99.204083),
    (19.444639, -99.204944),
    (19.446945, -99.207278),
    (19.444988, -99.210250),
    (19.446277, -99.210750),
    (19.445861, -99.211500),
]

def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(WIFI_SSID, WIFI_PASS)
    while not wlan.isconnected():
        print("Conectando a WiFi...")
        utime.sleep(1)
    print("WiFi conectado! IP:", wlan.ifconfig()[0])

def connect_mqtt():
    client_id = "ebike-" + str(utime.time())  # Genera un client_id único
    client = MQTTClient(client_id, MQTT_BROKER, port=MQTT_PORT, user=MQTT_TOKEN, password="", keepalive=30)

    try:
        client.connect()
        print(f"Conectado a MQTT como {client_id}")
    except Exception as e:
        print("Error conectando a MQTT:", str(e))
        return None
    return client

# Inicializar UART
uart = UART(1, baudrate=9600, tx=17, rx=16)

connect_wifi()
mqtt_client = connect_mqtt()

while True:
    if not mqtt_client:
        print("Reintentando conexión MQTT...")
        mqtt_client = connect_mqtt()
        if not mqtt_client:
            utime.sleep(2)
            continue

    for lat, lon in ROUTE:
        if not mqtt_client:
            print("Reintentando conexión MQTT...")
            mqtt_client = connect_mqtt()
            if not mqtt_client:
                utime.sleep(2)
                continue

        data = {
            "pgs_emea": {
                "latitude": lat,
                "longitude": lon,
                "timestamp": utime.time()
            }
        }
        gps_json = ujson.dumps(data)
        print("Publicando en MQTT:", gps_json)

        try:
            mqtt_client.publish(MQTT_TOPIC, gps_json, retain=False)
            mqtt_client.ping()  # Keep-alive para evitar desconexión
        except Exception as e:
            print("Error publicando MQTT:", str(e))
            mqtt_client = None  # Forzar reconexión

        utime.sleep(1)  # Reducir el tiempo de espera para evitar lentitud

```
# https://www.loom.com/share/f9501710188d4613bb9210b4e53e0e81?sid=508b3a03-5b00-48b1-96a4-63ff80bfe3d5
