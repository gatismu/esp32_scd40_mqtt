#include <Wire.h>
#include <SensirionI2CScd4x.h>
#include <WiFi.h>
#include <PubSubClient.h>

const char *ssid = "yr ssid";
const char *password = "passwd";

const char *mqtt_server = "192.168.1.111";
const int mqtt_port = 1883;
const char *mqtt_user = "user";
const char *mqtt_password = "passwd";
const char *mqtt_topic_temperature = "homeassistant/sensor/esp32iotsensor/CustomSensor_temp/state";
const char *mqtt_topic_humidity = "homeassistant/sensor/esp32iotsensor/CustomSensor_hum/state";
const char *mqtt_topic_co2 = "homeassistant/sensor/esp32iotsensor/CustomSensor_co2/state";

SensirionI2CScd4x scd4x;
WiFiClient espClient;
PubSubClient client(espClient);

uint16_t error;
float temperature;
float humidity;
uint16_t co2;

void callback(char *topic, byte *payload, unsigned int length) {
    // Handle MQTT messages if needed
}

void publishClient(const char *sensorName, const char *topic, const char *unit, const char *deviceClass, const char *uniqueId) {
    String configMessage = "{"
                           "\"name\":\"" + String(sensorName) + "\","
                           "\"state_topic\":\"" + String(topic) + "\","
                           "\"unit_of_measurement\":\"" + String(unit) + "\","
                           "\"device_class\":\"" + String(deviceClass) + "\","
                           "\"unique_id\":\"" + String(uniqueId) + "\""
                           "}";

    String topicString = String(topic) + "/config";
    const char *topicChar = topicString.c_str();
    const char *messageChar = configMessage.c_str();

    client.publish(topicChar, messageChar);
}

void reconnect() {
    while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");
        if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
            Serial.println("connected");

            publishClient("CustomSensor Temperature", mqtt_topic_temperature, "°C", "temperature", "CustomSensor_temp");
            publishClient("CustomSensor Humidity", mqtt_topic_humidity, "%", "humidity", "CustomSensor_hum");
            publishClient("CustomSensor CO2", mqtt_topic_co2, "ppm", "carbon_dioxide", "CustomSensor_co2");

            client.subscribe(mqtt_topic_temperature);
            client.subscribe(mqtt_topic_humidity);
            client.subscribe(mqtt_topic_co2);
        } else {
            Serial.print("failed, rc=");
            Serial.print(client.state());
            Serial.println(" try again in 5 seconds");
            delay(5000);
        }
    }
}

void setup() {
    Serial.begin(115200);
    while (!Serial) {
        delay(100);
    }

    Wire.begin();
    scd4x.begin(Wire);

    error = scd4x.stopPeriodicMeasurement();
    if (error) {
        Serial.print("Error trying to execute stopPeriodicMeasurement(): ");
        Serial.println(error);
    }

    error = scd4x.startPeriodicMeasurement();
    if (error) {
        Serial.print("Error trying to execute startPeriodicMeasurement(): ");
        Serial.println(error);
    }

    Serial.println("Waiting for first measurement... (5 sec)");

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);
    Serial.println("");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("");
    Serial.print("Connected to ");
    Serial.println(ssid);
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());

    client.setServer(mqtt_server, mqtt_port);
    client.setCallback(callback);
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    client.loop();

    bool isDataReady = false;
    error = scd4x.getDataReadyFlag(isDataReady);
    if (error) {
        Serial.print("Error trying to execute getDataReadyFlag(): ");
        Serial.println(error);
        return;
    }
    if (!isDataReady) {
        return;
    }
    error = scd4x.readMeasurement(co2, temperature, humidity);
    if (error) {
        Serial.print("Error trying to execute readMeasurement(): ");
        Serial.println(error);
    } else if (co2 == 0) {
        Serial.println("Invalid sample detected, skipping.");
    } else {
        Serial.print("CO2:");
        Serial.print(co2);
        Serial.print("\t");
        Serial.print("Temperature:");
        Serial.print(temperature);
        Serial.print("\t");
        Serial.print("Humidity:");
        Serial.println(humidity);

        // Publish MQTT data
        client.publish(mqtt_topic_temperature, String(temperature).c_str());
        client.publish(mqtt_topic_humidity, String(humidity).c_str());
        client.publish(mqtt_topic_co2, String(co2).c_str());
    }

    delay(30000); // 30-second delay between measurements
}
