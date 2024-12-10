# Visão Geral sobre Segurança em IoT
&nbsp;&nbsp;&nbsp; Nesta atividade proposta em sala pelo professor Filipe Resina, após uma aula sobre vulnerabilidades, riscos e contramedidas de segurança da informação em IoT, os alunos deviam realizar uma análise estática do código apresentado abaixo. O objetivo era identificar as vulnerabilidades presentes e compreender os tipos de ataques que poderiam explorá-las.

## Código fornecido
```cpp
/*********
  Rui Santos
  Complete project details at https://randomnerdtutorials.com  
*********/

// Load Wi-Fi library
#include <WiFi.h>

// Replace with your network credentials
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

// Set web server port number to 80
WiFiServer server(80);

// Variable to store the HTTP request
String header;

// Auxiliar variables to store the current output state
String output26State = "off";
String output27State = "off";

// Assign output variables to GPIO pins
const int output26 = 26;
const int output27 = 27;

// Current time
unsigned long currentTime = millis();
// Previous time
unsigned long previousTime = 0; 
// Define timeout time in milliseconds (example: 2000ms = 2s)
const long timeoutTime = 2000;

void setup() {
  Serial.begin(115200);
  // Initialize the output variables as outputs
  pinMode(output26, OUTPUT);
  pinMode(output27, OUTPUT);
  // Set outputs to LOW
  digitalWrite(output26, LOW);
  digitalWrite(output27, LOW);

  // Connect to Wi-Fi network with SSID and password
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  // Print local IP address and start web server
  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  server.begin();
}

void loop(){
  WiFiClient client = server.available();   // Listen for incoming clients

  if (client) {                             // If a new client connects,
    currentTime = millis();
    previousTime = currentTime;
    Serial.println("New Client.");          // print a message out in the serial port
    String currentLine = "";                // make a String to hold incoming data from the client
    while (client.connected() && currentTime - previousTime <= timeoutTime) {  // loop while the client's connected
      currentTime = millis();
      if (client.available()) {             // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        Serial.write(c);                    // print it out the serial monitor
        header += c;
        if (c == '\n') {                    // if the byte is a newline character
          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) {
            // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();
            
            // turns the GPIOs on and off
            if (header.indexOf("GET /26/on") >= 0) {
              Serial.println("GPIO 26 on");
              output26State = "on";
              digitalWrite(output26, HIGH);
            } else if (header.indexOf("GET /26/off") >= 0) {
              Serial.println("GPIO 26 off");
              output26State = "off";
              digitalWrite(output26, LOW);
            } else if (header.indexOf("GET /27/on") >= 0) {
              Serial.println("GPIO 27 on");
              output27State = "on";
              digitalWrite(output27, HIGH);
            } else if (header.indexOf("GET /27/off") >= 0) {
              Serial.println("GPIO 27 off");
              output27State = "off";
              digitalWrite(output27, LOW);
            }
            
            // Display the HTML web page
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<link rel=\"icon\" href=\"data:,\">");
            // CSS to style the on/off buttons 
            // Feel free to change the background-color and font-size attributes to fit your preferences
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");
            
            // Web Page Heading
            client.println("<body><h1>ESP32 Web Server</h1>");
            
            // Display current state, and ON/OFF buttons for GPIO 26  
      
```
## Primeira falha
&nbsp;&nbsp;&nbsp; Utilizando o script em bash abaixo, foi encontrada uma vulnerabilidade no código e no hardware, que ocorre devido a falta de gerenciamento de requisições. 

&nbsp;&nbsp;&nbsp; Primeiramente foi testado um ataque DoS à camada HTTP, porém o gerenciamento de requisições por fila padrão do protocolo fizeram com que o tester ainda sim conseguisse fazer suas requições. Após a primeira tentativa, o grupo testou um ataque SYN Flood Attack à camada TCP que não foi capaz de gerenciar a quantidade de requisições feitas pelo atacante fazendo com que o servidor não fosse capaz de processar as requisições do tester .

&nbsp;&nbsp;&nbsp; Por meio desses ataques concluisse que há uma vulnerabilidade no código devido a falta do gerenciamento eficaz de requisições por parte da camada TCP. Essa falha pode ser mitigada com a implementação de medidas como SYN Cookies, firewalls ou balanceamento de carga.

**Código bash**
```bash
sudo hping3 -S -p 80 --flood --rand-source 192.168.114.88
```

**Explicação do código**

**hping3**  uma ferramenta linux para explorar envio de pacotes

**-S** : Define o flag SYN

**-p** : Porta (80 - HTTP)

**--flood**: Envia pacotes o mais rápido possível. Não exibe respostas.

**--rand-souce**: Modo de endereço de origem aleatório

## Segunda falha

&nbsp;&nbsp;&nbsp; O código não possui nenhum tipo de autenticação, dessa forma, qualquer pessoa consegue se conectar a rede e enviar dados ao ESP32, alterando o estado dos leds da forma que preferir, sem precisar se registrar ou ser autorizado.

&nbsp;&nbsp;&nbsp; Essa falha pode causar diversos problemas, como o envio de dados maliciosos que podem gerar um comprometimento no ESP32, causando o mal funcionamento da aplicação.

&nbsp;&nbsp;&nbsp; Isso pode ser resolvido através de técnicas mais elevadas de autenticação, com a utilização de senhas e criptografia, garantindo o bom funcionamento do sistema.

## Extras
&nbsp;&nbsp;&nbsp; Para melhor entendimento do trabalho o grupo registrou prints do código executado e fotos do sistema de hardware utilizado como mostrado abaixo:

**Hardware**
![Hardware](assets/WhatsApp%20Image%202024-12-10%20at%2015.33.31%20(1).jpeg)
![Led amarelo](assets/WhatsApp%20Image%202024-12-10%20at%2015.33.30.jpeg)
![LEd azul](assets/WhatsApp%20Image%202024-12-10%20at%2015.33.31.jpeg)
![Led azul e amarelo](assets/WhatsApp%20Image%202024-12-10%20at%2015.33.08.jpeg)

**Código compilado**
![Código parte 1](assets/WhatsApp%20Image%202024-12-10%20at%2015.36.51.jpeg)
![Código parte 2](assets/WhatsApp%20Image%202024-12-10%20at%2015.36.47.jpeg)
![Código compilado](assets/WhatsApp%20Image%202024-12-10%20at%2015.36.42.jpeg)

