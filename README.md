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
