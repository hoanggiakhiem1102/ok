 /*  
  * Chương trình thử nghiệm sử dụng cảm biến 1 chạm để bật tắt led qua relay  
  * Dùng D1 làm chân nhận tín hiệu từ cảm biến chạm.  
  * Dùng D8 làm tín hiệu cấp cho relay  
  * Lưu ý:  
  *   - Khi chạm vào cảm biến thì giá trị nhận được luôn là 0.  
  *   - Khi không chạm cảm biến thì giá trị nhận được luôn là 1.  
  *   
  */  
   
 #include <ESP8266WiFi.h>   
 #include "HTTPSRedirect.h"  
 //#include "DebugMacros.h"  
 const char* ssid = "YourWifi";   
 const char* password = "Password";  
   
 //Google Script ID  
 const char *GScriptId = "GoogleScriptID";  
   
 //Khai báo thông tin google  
 const char* host = "script.google.com";   
 const char *googleRedirHost = "script.googleusercontent.com";  
 const int httpsPort = 443;   
 HTTPSRedirect client(httpsPort);  
   
 // Khai báo thông tin google sheet script đã publish  
 String url = String("/macros/s/") + GScriptId + "/exec?";  
   
 // PWM pin  
 int fanPIN = D5;  
   
 //  
 unsigned long preMillis = 0;  
 const long interval = 1000; // 5 seconds = 5 * 1000  
 int i = 0;  
   
 void setup() {  
   
  Serial.begin(115200);  
   
  // Serial.println("Khởi tạo giá trị "+String(initial));  
  initWifi();  
  pinMode(fanPIN, OUTPUT);  
  analogWrite(fanPIN,0);  
  delay(500);  
   
 }  
 int value = 0;  
 void loop() {  
 unsigned long currentMillis = millis();   
   
  if ( currentMillis - preMillis >= interval ){  
   preMillis = currentMillis;  
   if (WiFi.status() == WL_CONNECTED){  
       value = getData();  
       if( value != i){  
        i = value;  
        switch (i){  
         case 1: //speed 1  
          Serial.println(String(i));  
          analogWrite(fanPIN,768); //25%  
          break;  
         case 2: //speed 2  
          Serial.println(String(i));  
          analogWrite(fanPIN,853); //50%  
          break;  
         case 3: //speed 3  
          Serial.println(String(i)); //75%  
          analogWrite(fanPIN,938);   
          break;  
         case 4: //speed 4  
          Serial.println(String(i)); //100%  
          analogWrite(fanPIN,1023);  
          break;  
         default:  
          Serial.println("0"); //0%  
          analogWrite(fanPIN,0);  
          break;  
        }  
       }  
    }  
  }  
 }  
   
   
   
 void initWifi(){  
   Serial.println("Connecting to wifi: ");   
   Serial.println(ssid);   
   Serial.flush();  
   WiFi.begin(ssid, password);   
   while (WiFi.status() != WL_CONNECTED) {   
       delay(500);   
       Serial.print(".");   
   }   
   Serial.print(" IP address: ");   
   Serial.println(WiFi.localIP());  
   
   // Use HTTPSRedirect class to create a new TLS connection  
   client.setInsecure();  
   client.setPrintResponseBody(false);  
   client.setContentTypeHeader("application/json");  
   
     
   Serial.print(String("Connecting to "));   
   Serial.println(host);  
   bool flag = false;   
   for (int i=0; i<5; i++){   
       int retval = client.connect(host, httpsPort);   
       if (retval == 1) {   
             flag = true;   
             break;   
       }   
       else   
           Serial.println("Connection failed. Retrying…");   
   }  
   
   // Connection Status, 1 = Connected, 0 is not.   
   Serial.println("Connection Status: " + String(client.connected()));   
   Serial.flush();   
      
   if (!flag){   
       Serial.print("Could not connect to server: ");   
       Serial.println(host);   
       Serial.println("Exiting…");   
       Serial.flush();   
       return;   
   }  
 }  
   
 void postData(String ledStatus){  
   if (!client.connected()){   
       Serial.println("Connecting to client again…");   
       client.connect(host, httpsPort);   
   }  
   if(ledStatus == "1"){  
    ledStatus = "off";  
   }else{  
    ledStatus = "on";  
   }  
   String urlFinal = url + "ledStatus=" + ledStatus;   
   client.POST(urlFinal, host, "");   
 }  
   
 int getData(){  
  String str = "";  
  String urlGET = "";  
  if (!client.connected()){   
       Serial.println("Connecting to client again…");   
       client.connect(host, httpsPort);   
  }  
  client.GET(url, host);  
  str = client.getResponseBody();  
  return str.toInt();  
 }  
