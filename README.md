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

server.on("/update", HTTP_GET, []() {
  String num = server.arg("num");
  if (num.length() == 1 && puzzleIndex < 4) {
    receivedCode[puzzleIndex] = num[0];
    digitalWrite(segmentPins[puzzleIndex], HIGH);
    puzzleIndex++;
  }
  server.send(200, "text/plain", receivedCode);
});
const char htmlPage[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="he">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>מערכת נעילה אלקטרונית</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      text-align: center;
      padding: 20px;
    }
    h1 {
      color: #333;
    }
    input {
      font-size: 20px;
      padding: 10px;
      width: 200px;
      margin: 10px;
    }
    button {
      padding: 10px 20px;
      font-size: 20px;
      background-color: #28a745;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background-color: #218838;
    }
    #message {
      margin-top: 20px;
      font-size: 18px;
      color: red;
    }
  </style>
</head>
<body>
  <h1>הזן את קוד הנעילה</h1>
  <input type="password" id="code" maxlength="4" placeholder="----">
  <button onclick="sendCode()">שלח</button>
  <div id="message"></div>
  <script>
    function sendCode() {
      var code = document.getElementById('code').value;
      if(code.length !== 4){
        document.getElementById('message').innerHTML = '⚠️ יש להזין 4 ספרות';
        return;
      }
      fetch('/unlock?code=' + code)
        .then(response => response.text())
        .then(data => {
          document.getElementById('message').innerHTML = data;
        })
        .catch(err => {
          document.getElementById('message').innerHTML = 'שגיאה: ' + err;
        });
    }
  </script>
</body>
</html>
)rawliteral";

server.on("/", HTTP_GET, []() {
  server.send_P(200, "text/html", htmlPage);
});

server.on("/unlock", HTTP_GET, []() {
  if (server.arg("code") == correctCode) {
    digitalWrite(lockPin, LOW); // פתיחת הדלת
    server.send(200, "text/html", "<h1>Door Unlocked!</h1>");
  } else {
    server.send(200, "text/html", "<h1>Wrong Code!</h1>");
  }
});

void sendNumberToLock(char num) {
  HTTPClient http;
  String url = String(lockServer) + "?num=" + num;
  http.begin(client, url);
  http.GET();
  http.end();
}
void solvePuzzle1() {
  int brightness = analogRead(lightSensor);
  if (brightness < 800) {
    delay(2000);
    sendNumberToLock(puzzleCode[puzzleIndex++]);
  }
}

void solvePuzzle2() {
  int temperature = analogRead(tempSensor);
  if (temperature < 500) {
    delay(2000);
    sendNumberToLock(puzzleCode[puzzleIndex++]);
  }
}

void solvePuzzle3() {
  bool sequenceCorrect = true;
  for (int i = 0; i < 4; i++) {
    digitalWrite(ledPins[i], HIGH);
    delay(500);
    digitalWrite(ledPins[i], LOW);
    if (!digitalRead(buttonPins[i])) {
      sequenceCorrect = false;
    }
  }
  if (sequenceCorrect) {
    sendNumberToLock(puzzleCode[puzzleIndex++]);
  }
}

void solvePuzzle4() {
  int distance = analogRead(distanceSensor);
  if (distance > 700) {
    delay(2000);
    sendNumberToLock(puzzleCode[puzzleIndex++]);
  }
}

void loop() {
  server.handleClient(); // מאזין לבקשות HTTP מהמשתמשים
  // קריאת הפונקציות לבדיקת חידות:
  solvePuzzle1();
  solvePuzzle2();
  solvePuzzle3();
  solvePuzzle4();
}
.
