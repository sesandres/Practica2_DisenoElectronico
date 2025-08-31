```
#include "UbidotsEsp32Mqtt.h"
#include <TFT_eSPI.h>
#include <SPI.h>

// === Configuración Ubidots y Wi-Fi ===
const char *UBIDOTS_TOKEN = "token";  // Tu token
const char *WIFI_SSID = "wifi";
const char *WIFI_PASS = "password";
const char *DEVICE_LABEL = "esp23";    // Nombre del dispositivo en Ubidots
const char *VARIABLE_LABEL = "sw1";    // Nombre de la variable en Ubidots

#define RELAY_PIN 15  // Pin conectado al relé

Ubidots ubidots(UBIDOTS_TOKEN);
TFT_eSPI tft = TFT_eSPI(); 

void drawRelayState(bool relayState) {
  tft.fillScreen(TFT_BLACK);           
  tft.setTextSize(2);
  tft.setTextDatum(MC_DATUM);         
  tft.setTextColor(TFT_WHITE);

  if (relayState) {
    // Círculo verde con "ON"
    tft.fillCircle(130, 68, 60, TFT_GREEN);
     tft.fillCircle(130, 68, 50, TFT_BLACK);
     tft.fillCircle(130, 68, 35, TFT_GREEN);
    //tft.drawCentreString("ON", 140, 80, 2);
  } else {
    // Círculo azul con "OFF"
    tft.fillCircle(130, 68, 60, TFT_BLUE);
     tft.fillCircle(130, 68, 50, TFT_BLACK);
     tft.fillCircle(130, 68, 35, TFT_BLUE);
  }
}

/****************************************
 * Función de Callback MQTT
 ****************************************/
void callback(char *topic, byte *payload, unsigned int length) {
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }

  Serial.print("Dato recibido: ");
  Serial.println(message);

  float value = message.toFloat();

  if (value == 1.0) {
    digitalWrite(RELAY_PIN, HIGH);
    Serial.println("Relé ENCENDIDO");
    drawRelayState(true);
  } else {
    digitalWrite(RELAY_PIN, LOW);
    Serial.println("Relé APAGADO");
    drawRelayState(false);
  }
}

/****************************************
 * Configuración
 ****************************************/
void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW);  // Estado inicial: apagado

  // Inicializar pantalla
  tft.init();
  tft.setRotation(1);
  tft.fillScreen(TFT_BLACK);
  drawRelayState(false);  // Mostrar estado inicial

  // Conexión a Wi-Fi y Ubidots
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS);
  ubidots.setCallback(callback);
  ubidots.setup();
  ubidots.reconnect();
  ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL);
}

/****************************************
 * Bucle principal
 ****************************************/
void loop() {
  if (!ubidots.connected()) {
    ubidots.reconnect();
    ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL);
  }
  ubidots.loop();
}
```