## Introdução
Para verificar o funcionamento do Módulo ESP32, foram realizados alguns testes com código básicos

### Código 1 - WIFI Scan
Esse código realiza o escaneamento das Redes Wifi que estão no Alcance do ESP32, e comunica via Serial

```cpp
#include "WiFi.h"

void setup() {
  Serial.begin(115200);

  // Set WiFi to station mode and disconnect from an AP if it was previously connected.
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(100);

  Serial.println("Setup done");
}

void loop() {
  Serial.println("Scan start");

  // WiFi.scanNetworks will return the number of networks found.
  int n = WiFi.scanNetworks();
  Serial.println("Scan done");
  if (n == 0) {
    Serial.println("no networks found");
  } else {
    Serial.print(n);
    Serial.println(" networks found");
    Serial.println("Nr | SSID                             | RSSI | CH | Encryption");
    for (int i = 0; i < n; ++i) {
      // Print SSID and RSSI for each network found
      Serial.printf("%2d", i + 1);
      Serial.print(" | ");
      Serial.printf("%-32.32s", WiFi.SSID(i).c_str());
      Serial.print(" | ");
      Serial.printf("%4ld", WiFi.RSSI(i));
      Serial.print(" | ");
      Serial.printf("%2ld", WiFi.channel(i));
      Serial.print(" | ");
      switch (WiFi.encryptionType(i)) {
        case WIFI_AUTH_OPEN:            Serial.print("open"); break;
        case WIFI_AUTH_WEP:             Serial.print("WEP"); break;
        case WIFI_AUTH_WPA_PSK:         Serial.print("WPA"); break;
        case WIFI_AUTH_WPA2_PSK:        Serial.print("WPA2"); break;
        case WIFI_AUTH_WPA_WPA2_PSK:    Serial.print("WPA+WPA2"); break;
        case WIFI_AUTH_WPA2_ENTERPRISE: Serial.print("WPA2-EAP"); break;
        case WIFI_AUTH_WPA3_PSK:        Serial.print("WPA3"); break;
        case WIFI_AUTH_WPA2_WPA3_PSK:   Serial.print("WPA2+WPA3"); break;
        case WIFI_AUTH_WAPI_PSK:        Serial.print("WAPI"); break;
        default:                        Serial.print("unknown");
      }
      Serial.println();
      delay(10);
    }
  }
  Serial.println("");

  // Delete the scan result to free memory for code below.
  WiFi.scanDelete();

  // Wait a bit before scanning again.
  delay(5000);
}
```
### Código 2 - Funcionamento do Motor
Assim como havia sido comentado anteriormente pelos professores, precisávamos verificar o motivo pelo qual o motor não estava funcionando. Se o problema era no próprio motor ou na programação. Para isso, foi criado um código que testa não somente a potência do motor, mas também a inversão de sentido.

```cpp
// Define os pinos do ESP32 conectados ao L298N
#define IN1 12  // Pino IN1 do L298N conectado ao GPIO25 do ESP32 
#define IN2 14  // Pino IN2 do L298N conectado ao GPIO26 do ESP32 
#define ENA 13  // Pino ENA (Enable A) do L298N conectado ao GPIO33 do ESP32 

void setup() {
  // Configura os pinos como saída
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENA, OUTPUT);

  // Inicia o motor parado
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0); // PWM de 0% (motor desligado)
}

void loop() {
  // Motor gira para frente
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 128); // PWM de 50% (ajuste conforme necessário)
  delay(5000);           // Aguarda 5 segundos

  // Para o motor
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0);
  delay(2000);           // Aguarda 2 segundos

  // Motor gira para trás
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  analogWrite(ENA, 200); // PWM de 78% (ajuste conforme necessário)
  delay(5000);           // Aguarda 5 segundos

  // Para o motor novamente
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0);
  delay(2000);           // Aguarda 2 segundos
}
```

### Código 3 - Funcionamento da Comunicação MQTT
Como estamos trabalhando agora com o grupo responsável pelo Home Assistant, começamos a testar a Comunicação MQTT que vai ocorrer entre o ESP32 e o Broker. A comunicação ocorreu conforme planejado

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// Configurações Wi-Fi
const char *ssid = "lpae_wifi";
const char *password = "esp-8266";

// Configurações do MQTT
const char *mqtt_broker = "192.168.1.2";
const char *topic = "v1/devices/me/telemetry"; // Tópico de publicação
const int mqtt_port = 1883;
const char *mqtt_user = "76HUc0AcMh9ZXR3fql2n"; // Nome de usuário
const char *mqtt_password = "";                // Senha (se necessário)

// Variáveis para MQTT
WiFiClient espClient;
PubSubClient client(espClient);

// Função para conectar ao Wi-Fi
void setup_wifi() {
  delay(10);
  Serial.print("Conectando ao WiFi");
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi conectado!");
  Serial.print("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

// Função para conectar ao broker MQTT
void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentando conectar ao MQTT...");
    String client_id = "esp32-client-";
    client_id += String(WiFi.macAddress());

    if (client.connect(client_id.c_str(), mqtt_user, mqtt_password)) {
      Serial.println("Conectado ao broker MQTT!");
    } else {
      Serial.print("Falha ao conectar. Estado: ");
      Serial.println(client.state());
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  setup_wifi();

  client.setServer(mqtt_broker, mqtt_port);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Publica um JSON no tópico especificado
  static unsigned long lastPublishTime = 0;
  if (millis() - lastPublishTime > 5000) { // Publica a cada 5 segundos
    String message = "{\"id\": 49, \"cor\": \"vermelho\"}"; // JSON formatado corretamente
    client.publish(topic, message.c_str());
    Serial.println("Mensagem publicada no tópico v1/devices/me/telemetry: " + message);
    lastPublishTime = millis();
  }
}
```

### Código 4 - Código do Motor e Sensor de Velocidade
Assim como comentado no Design, buscamos realizar as melhorias que tinhamos como metas no próprio código unificado que iria ser compilado no ESP32. Entre as melhorias temos a realização do cálculo da Velocidade no Formato de m/s, a Comunicação do MQTT em Mensagens no formato .json, para facilitar o recebimento das informações pelo Home Assistant

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include "acelera.h"
#include "sensor.h"

// Configurações Wi-Fi
const char *ssid = "lpae_wifi";
const char *password = "esp-8266";

// Configurações MQTT
const char *mqtt_broker = "192.168.1.2";
const char *topic = "v1/devices/me/telemetry";
const int mqtt_port = 1883;
const char *mqtt_user = "76HUc0AcMh9ZXR3fql2n";
const char *mqtt_password = "";
WiFiClient espClient;
PubSubClient client(espClient);

// Gerenciamento de tempo
unsigned long currentTime;
unsigned long intervalRPM = 1000;
unsigned long intervalControl = 100;
unsigned long lastTimeRPM;
unsigned long lastTimeControl;

// Variáveis do motor
unsigned int velocidade = 0;        // Velocidade em m/s
const float v_max = 10.0;           // Velocidade máxima em m/s
unsigned int rampa = 5000;
bool sentido = true;
bool liga = false;
unsigned int PWM_atual = 0;

// Variáveis para cálculo de RPM
volatile unsigned int rpmCount = 0; // Contador de pulsos
const int sensorPin = 34;           // Pino do sensor de RPM

// Função de interrupção para contar pulsos
void countRPM() {
    rpmCount++;
}

// Conecta ao Wi-Fi
void setup_wifi() {
    delay(10);
    Serial.print("Conectando ao WiFi");
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("Conectado ao WiFi");
}

// Conecta ao servidor MQTT
void reconnect() {
    while (!client.connected()) {
        Serial.print("Tentando conectar ao MQTT...");
        if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
            Serial.println("Conectado ao MQTT");
            client.subscribe(topic);
        } else {
            Serial.print("Falha na conexão, rc=");
            Serial.println(client.state());
            delay(5000);
        }
    }
}

// Envia mensagens JSON
void sendMessage(const char *sensor, const char *assunto, float valor) {
    StaticJsonDocument<256> doc;
    doc["sensor"] = sensor;
    doc["assunto"] = assunto;
    doc["valor"] = valor;

    char buffer[256];
    size_t n = serializeJson(doc, buffer);
    client.publish(topic, buffer, n);
}

// Callback para processar mensagens recebidas
void Callback(char *topic, byte *payload, unsigned int length) {
    Serial.print("Mensagem recebida no tópico: ");
    Serial.println(topic);

    String msg;
    for (int i = 0; i < length; i++) {
        msg += (char)payload[i];
    }
    Serial.print("Mensagem: ");
    Serial.println(msg);

    StaticJsonDocument<256> doc;
    DeserializationError error = deserializeJson(doc, msg);

    if (error) {
        Serial.println("Erro ao analisar JSON!");
        return;
    }

    const char *sensor = doc["sensor"];
    const char *assunto = doc["assunto"];
    float valor = doc["valor"];

    if (strcmp(sensor, "motor") == 0) {
        if (strcmp(assunto, "liga") == 0) {
            liga = valor;
            Serial.println(liga ? "Motor ligado" : "Motor desligado");
        } else if (strcmp(assunto, "velocidade") == 0) {
            velocidade = constrain(valor, 0, v_max); // Velocidade em m/s limitada ao máximo
            Serial.printf("Velocidade ajustada para: %.2f m/s\n", velocidade);
        } else if (strcmp(assunto, "rampa") == 0) {
            rampa = max(valor, 0.0f);
            Serial.printf("Rampa ajustada para: %d ms\n", rampa);
        } else if (strcmp(assunto, "sentido") == 0) {
            sentido = valor;
            Serial.println(sentido ? "Sentido: Normal" : "Sentido: Inverso");
        }
    }
}

// Calcula e envia o RPM
void CalcRPM() {
    currentTime = millis();
    if (currentTime - lastTimeRPM >= intervalRPM) {
        float rps = rpmCount * 1000.0 / (20.0 * (currentTime - lastTimeRPM));
        rpmCount = 0;

        sendMessage("motor", "RPM", rps * 60);

        lastTimeRPM = millis();
    }
}

// Controla o motor
void MotorControl(bool liga, unsigned int &PWM, float velocidade, unsigned int rampa = 5000, bool sentido = true) {
    currentTime = millis();

    // Converte a velocidade em m/s para um valor de PWM proporcional
    unsigned int PWM_alvo = (velocidade / v_max) * 255; 

    if (liga && (PWM_alvo != PWM)) {
        PWM = acelera(PWM, PWM_alvo, rampa);
    } else if (!liga && PWM != 0) {
        PWM = acelera(PWM, 0, 2000);
    }
}

void setup() {
    Serial.begin(115200);
    Serial.println("Início");

    pinMode(sensorPin, INPUT_PULLUP);
    setup_wifi();

    client.setServer(mqtt_broker, mqtt_port);
    client.setCallback(Callback);

    attachInterrupt(digitalPinToInterrupt(sensorPin), countRPM, RISING);
    currentTime = millis();
    lastTimeRPM = currentTime;
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    client.loop();
    MotorControl(liga, PWM_atual, velocidade, rampa, sentido);
    CalcRPM();
}
```

### Código 5 - Código do Sensor de Cor, para aumentar a gama de cores identificáveis
O código do Sensor de Cor estava funcionando muito bem, a única melhoria que realizei foi de aumentar a gama de Cores que o sensor é capaz de identificar com base na lógica condicional.

```cpp
#include <Wire.h>
#include "Adafruit_TCS34725.h"
#include <WiFi.h>
#include <PubSubClient.h>

// Configurações do sensor de presença
const int presenceSensorPin = 27; // Pino conectado ao sensor de presença (D27 no ESP-32)

// Criação do objeto para o sensor TCS34725 com tempo de integração e ganho definidos
Adafruit_TCS34725 tcs = Adafruit_TCS34725(TCS34725_INTEGRATIONTIME_50MS, TCS34725_GAIN_4X);

bool objectPreviouslyDetected = false; // Variável para rastrear o estado anterior do objeto

// Configurações Wi-Fi
const char *ssid = "lpae_wifi";
const char *password = "esp-8266";

// Configurações MQTT
const char *mqtt_broker = "192.168.1.2";
const char *topic = "v1/devices/me/telemetry";
const int mqtt_port = 1883;
const char *mqtt_user = "76HUc0AcMh9ZXR3fql2n";
const char *mqtt_password = "";

WiFiClient espClient;
PubSubClient client(espClient);

// Função para conectar ao WiFi
void setup_wifi() {
    Serial.println("Conectando ao WiFi...");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nWiFi conectado!");
}

// Função para conectar ao MQTT
void reconnect() {
    while (!client.connected()) {
        Serial.print("Conectando ao MQTT...");
        if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
            Serial.println("Conectado ao MQTT!");
            client.subscribe(topic);
        } else {
            Serial.print("Falha na conexão, rc=");
            Serial.println(client.state());
            delay(2000);
        }
    }
}

// Função para enviar mensagens JSON ao MQTT
void sendMessage(const char *sensor, const char *assunto, const char *valor) {
    char buffer[256];
    snprintf(buffer, sizeof(buffer), "{\"sensor\": \"%s\", \"assunto\": \"%s\", \"valor\": \"%s\"}", sensor, assunto, valor);
    client.publish(topic, buffer);
}

void setup() {
    // Inicializa a comunicação serial
    Serial.begin(115200);
    pinMode(presenceSensorPin, INPUT);

    // Inicializa o sensor TCS34725
    if (tcs.begin()) {
        Serial.println("Sensor TCS34725 encontrado!");
    } else {
        Serial.println("Sensor TCS34725 não encontrado. Verifique as conexões.");
        while (1);
    }

    // Configura WiFi e MQTT
    setup_wifi();
    client.setServer(mqtt_broker, mqtt_port);
    client.setCallback([](char *topic, byte *payload, unsigned int length) {
        Serial.print("Mensagem recebida no tópico: ");
        Serial.println(topic);
    });
}

unsigned long lastColorCheck = 0; // Armazena o tempo da última leitura
const unsigned long interval = 3000; // Intervalo de 10 segundos

void loop() {
    // Mantém a conexão MQTT
    if (!client.connected()) {
        reconnect();
    }
    client.loop();

    // Verifica a presença do objeto
    bool currentPresence = digitalRead(presenceSensorPin) == HIGH;

    // Lê a cor a cada 10 segundos, se houver um objeto presente
    if (currentPresence && millis() - lastColorCheck >= interval) {
        lastColorCheck = millis(); // Atualiza o tempo da última leitura

        // Lê os valores de cor
        uint16_t red, green, blue, clear;
        tcs.getRawData(&red, &green, &blue, &clear);

        // Evita divisão por zero
        if (clear == 0) clear = 1;

        // Normalização dos valores de cor
        float r = (float)red / clear;
        float g = (float)green / clear;
        float b = (float)blue / clear;

        // Converte para o formato hexadecimal
        int rHex = (int)(r * 255.0);
        int gHex = (int)(g * 255.0);
        int bHex = (int)(b * 255.0);

        char hexColor[8];
        snprintf(hexColor, sizeof(hexColor), "#%02X%02X%02X", rHex, gHex, bHex);

        // Publica a cor detectada no tópico MQTT
        Serial.printf("Cor detectada: %s\n", hexColor);
        sendMessage("sensor_de_cor", "cor", hexColor);
    }

    delay(100); // Pequeno atraso para evitar processamento excessivo
}
```
