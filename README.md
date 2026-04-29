#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "WXHBXK";
const char* password = "258258258";

String GAS_URL="https://script.google.com/macros/s/AKfycbylwXelzwEYboeGOrqHal-mTySMdTU1cmaRKuKQM9m7DnrCXKxw0slxVHeDRabqZbHq/exec";


#define WATER_SENSOR 27
#define LED 23
#define BUZZER 19

#define TRIG 5
#define ECHO 18

long duration;
float distance;

bool systemEnable = true;


void setup() {

Serial.begin(115200);

pinMode(WATER_SENSOR, INPUT);

pinMode(LED, OUTPUT);
pinMode(BUZZER, OUTPUT);

pinMode(TRIG, OUTPUT);
pinMode(ECHO, INPUT);


Serial.println("Connecting WiFi...");

WiFi.begin(ssid,password);

while(WiFi.status()!=WL_CONNECTED){
delay(500);
Serial.print(".");
}

Serial.println();
Serial.println("WiFi Connected");
Serial.println(WiFi.localIP());

}



void loop() {


// =========================
// Ultrasonic
// =========================

digitalWrite(TRIG, LOW);
delayMicroseconds(2);

digitalWrite(TRIG, HIGH);
delayMicroseconds(10);

digitalWrite(TRIG, LOW);

duration = pulseIn(ECHO, HIGH);

distance = duration * 0.034 / 2;


// =========================
// Water Sensor
// =========================

int water = digitalRead(WATER_SENSOR);


// =========================
// Alarm Logic
// =========================

if(systemEnable && water==0){

digitalWrite(LED,HIGH);
digitalWrite(BUZZER,HIGH);
delay(300);

digitalWrite(LED,LOW);
digitalWrite(BUZZER,LOW);
delay(300);

}
else{

digitalWrite(LED,LOW);
digitalWrite(BUZZER,LOW);

}



// =========================
// Send To Cloud
// =========================

if(WiFi.status()==WL_CONNECTED){

String fullURL=
GAS_URL+
"?distance="+String(distance)+
"&water="+String(water);


Serial.println("---------------");
Serial.println("Sending Data...");

Serial.print("Distance: ");
Serial.println(distance);

Serial.print("Water: ");
Serial.println(water);

Serial.println(fullURL);


HTTPClient http;

http.begin(fullURL);

http.setFollowRedirects(
HTTPC_STRICT_FOLLOW_REDIRECTS
);


int httpCode=http.GET();

Serial.print("HTTP Code: ");
Serial.println(httpCode);


if(httpCode>0){

String command=http.getString();

command.trim();

Serial.print("Cloud Command: ");
Serial.println(command);


// รับคำสั่งจาก Google Sheet D2
if(command=="OFF"){
systemEnable=false;
Serial.println("SYSTEM DISABLED");
}

if(command=="ON"){
systemEnable=true;
Serial.println("SYSTEM ENABLED");
}

Serial.println("Send Success");

}
else{

Serial.print("Error: ");
Serial.println(
http.errorToString(httpCode).c_str()
);

}

http.end();

}
else{

Serial.println("WiFi Disconnected");

}


delay(500);

}
