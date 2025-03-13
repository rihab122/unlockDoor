#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <ESP8266HTTPClient.h>

const char* ssid = "YourWiFi";
const char* password = "YourPassword";

ESP8266WebServer server(80);
const int lockPin = D1;                          
const int segmentPins[] = {D2, D3, D4, D5};       
String correctCode = "1234";                     
String receivedCode = "----";

const char* lockServer = "http://<LOCK_ESP_IP>/update";  
const int lightSensor = A0;                      
const int tempSensor = D6;                       
const int ledPins[] = {D7, D8, D9, D10};          
const int buttonPins[] = {D3, D4, D5, D6};         
const int distanceSensor = A1;                   
int puzzleIndex = 0;                             
String puzzleCode = "1234";                      
WiFiClient client;
