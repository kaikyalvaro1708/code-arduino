```c++
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <ArduinoJson.h>
 
// CONFIGURAÇÕES DO WI-FI
const char* ssid = "Infinix HOT 11S NFC";
const char* password = "Kaiky&17";
 
// CONFIGURAÇÕES DO MQTT
const char* mqtt_server = "broker.hivemq.com"; // Broker público
const int mqtt_port = 1883;
const char* mqtt_user = ""; // Se necessário
const char* mqtt_pass = ""; // Se necessário
const char* mqtt_topic = "juan/dht11"; // Tópico MQTT
 
// SENSOR DHT
#define DHTPIN 4
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
 
WiFiClient espClient;
PubSubClient client(espClient);
 
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando-se ao WiFi: ");
  Serial.println(ssid);
 
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
 
  Serial.println("");
  Serial.println("WiFi conectado!");
  Serial.println("IP: ");
  Serial.println(WiFi.localIP());
}
 
void reconnect() {
  // Função reconectar deve ser chamada apenas quando necessário
  while (!client.connected()) {
    Serial.print("Conectando ao MQTT...");
    if (client.connect("ESP32Client", mqtt_user, mqtt_pass)) {
      Serial.println("conectado!");
    } else {
      Serial.print("falhou, rc=");
      Serial.print(client.state());
      Serial.println(" tentando novamente em 5 segundos...");
      delay(5000);
    }
  }
}
 
void setup() {
  Serial.begin(9600);
  dht.begin();
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
 
  // Conectar ao MQTT uma vez no início
  reconnect();
}
 
void loop() {
  // Apenas tente reconectar se a conexão MQTT for perdida
  if (!client.connected()) {
    reconnect();
  }
 
  client.loop(); // Processa qualquer mensagem MQTT recebida
 
  delay(2000); // Espera 2 segundos entre as leituras
 
  // Lê os valores do sensor
  float temperatura = dht.readTemperature();
  float umidade = dht.readHumidity();
 
  if (isnan(umidade) || isnan(temperatura)) {
    Serial.println("[MonteCarloDigital] ERRO: Falha na leitura do sensor DHT!");
    return;
  }
 
  // Cria JSON
  StaticJsonDocument<200> doc;
  doc["sensor"] = "DHT11";
  doc["temperatura"] = temperatura;
  doc["umidade"] = umidade;
 
  String json;
  serializeJson(doc, json);
 
  // Envia os dados para o broker MQTT
  client.publish(mqtt_topic, json.c_str());
 
  // Também imprime no Serial (opcional)
  Serial.print("MQTT >> ");
  Serial.println(json);
}
```
 
