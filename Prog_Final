//inserir bibliotecas
#include <WiFi.h>
#include <PubSubClient.h>
#include <max6675.h>

//Conexão a internet local
const char* ssid     = "NOME DO SEU WIFI";
const char* password = "SENHA DO SEU WIFI";
const char* mqtt_server = "BROKER MQTT UTILIZADO"; //BROKER MQTT

WiFiClient espClient;
PubSubClient client(espClient); // Indicando que estou utilizando a conexão Wifi para comunicar com o MQTT 
unsigned long lastMsg = 0; // variável será usada para rastrear o tempo da última mensagem enviada ou recebida pelo cliente MQTT.
char msg[50]; //variávelusada para armazenar mensagens recebidas do servidor MQTT, máximo de 50 caracteres
int value = 0;
String payload = ""; // Para armazenar a carga útil que será enviada ao MQTT

int thermoDO = 19;
int thermoCS = 23;
int thermoCLK = 5;
float temperatura;

MAX6675 thermocouple(thermoCLK, thermoCS, thermoDO);

void setup() {
  Serial.begin(9600);

  Serial.println("MAX6675 test");
  // Mensagem inicial para conexão e estabilização do sensor
  delay(500);
  
  // Enquanto tenta conexão com o Wifi
  Serial.print("Connecting to ");
  delay(10);
  Serial.println(ssid);

  // WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  
  // Conectado ao Wifi
  while(WiFi.status() != WL_CONNECTED){ 
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  //Serial.println("");
  Serial.println("Wifi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  // Apontando o link do broker e a porta de comunicação
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void callback(char* topic, byte* message, unsigned int length) {
 Serial.print("Message arrived on topic: ");
 Serial.print(topic);
 Serial.print(". Message: ");
 String messageTemp;

  for (int i = 0; i < length; i++){
    Serial.print((char)message[1]);
    messageTemp += (char)message[i];
  }

  if(String(topic) == "tacho/publisher"){
    if(messageTemp == "on"){
      Serial.println("on");
    }

    else if(messageTemp == "off"){
      Serial.println("Off");
    }
  }
}

void reconnect() {
  
  // Tentativa de reconexão ao broker MQTT
  while (!client.connected()) {
  Serial.print("Attempting MQTT connection...");
  String clientId = "TACHO_MQTT";
  clientId += String (random(0xffff), HEX);

  if (client.connect(clientId.c_str())) {
  Serial.println("connected");

      client.subscribe("tacho/publisher");
    } else {
  Serial.print("failed, rc=");
  Serial.print(client.state());
  Serial.println(" try again in 5 seconds");
      delay(5000); // Aguarde 5s para tentar novamente;
    }
  }
}


void loop() {

// Quando conectado ao Wifi e ao broker MQTT
if (!client.connected()) {
    reconnect();
  } 

  client.loop();
  temperatura = thermocouple.readCelsius();
  Serial.println(thermocouple.readCelsius()); 
    payload = temperatura; // Cria uma string JSON contendo a temperatura, isso para enviar o MQTT  
    client.publish("tacho/temperatura", payload.c_str()); // Publica a temperatura no tópico MQTT "tacho/temperatura".
  delay(1000);
}
