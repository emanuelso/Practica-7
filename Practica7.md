César Roldan Moros  
*Grup 13*
____
# PRÁCTICA 7: Bus de comunicaciones III
##### Objetivo
El objetivo de esta práctica es entender el funcionamiento del bus I2C y realizar una práctica para entender su funcionamiento
___
### Ejercicio 1. Reproducción desde memória interna
##### Fotos del montaje
Las fotos y videos del montaje se encuentran en el mismo repositorio de la práctica en el github
##### Codigo
```
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"
#include "FS.h"
#include "HTTPClient.h"
#include "SPI.h"
#include "SD.h"
#include "SPIFFS.h"
AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;
void setup(){
  Serial.begin(115200);
  in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac)); 
  aac = new AudioGeneratorAAC();
  out = new AudioOutputI2S();
  out -> SetGain(0.125);
  out -> SetPinout(26,25,22); 
  aac->begin(in, out); 
}
void loop(){
  if (aac->isRunning()) { 
    aac->loop();
  } 
  else { // si no
    aac -> stop();
    Serial.printf("Sound Generator\n"); 
    delay(1000);
  }
}
```
##### Funcionamiento
Primero agregamos algunos de los archivos de cabecera de la librería ESP8266.
Los datos del sonido digital se almacenan en el archivo de la cabecera sampleaac.h. Después damos a los tres #includes una variable más corta que contiene funciones.
En el void setup() establecemos una velocidad de transmisión de 115200 bauds e inicializamos los archivos de la cabecera. Por *AudioFileSourcePROGMEM* definimos que el archivo de audio de muestra es el sampleaac dentro de una matriz.
El objeto AudioOutputI2S tiene diferentes funciones.
Utilizamos la función SetGain para reducir el volumen del altavoz y definimos el pinout con la función SetPinout
Ahora conectamos los datos del sonido de entrada desde la memoria interna del programa a la salida de audio I2S con la función
*AudioGeneratorAAC*.
En el void loop(), el generador de audio sigue funcionando hasta que toda la matriz del sonido pasa por el generador. Cuando el generador está listo, deja de funcionar y en la salida del Serial podremos ver que el generador de sonido está preparado y listo

____
### Ejercicio 2: Reproducir un archivo WAVE en el ESP32 desde una tarjeta SD externa

##### Codigo
```
#include "Audio.h"
#include "SD.h"
#include "FS.h"
#define SD_CS 5
#define SPI_MOSI 23
#define SPI_MISO 19
#define SPI_SCK 18
#define I2S_DOUT 25
#define I2S_BCLK 27
#define I2S_LRC 26
Audio audio;
void setup(){
  pinMode(SD_CS, OUTPUT);
  digitalWrite(SD_CS, HIGH);
  SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI); 
  Serial.begin(115200);
  SD.begin(SD_CS); 
  audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT); 
  audio.setVolume(10); 
  audio.connecttoFS(SD, "Ensoniq-ZR-76-01-Dope-77.wav"); 
}
void loop()
{
	audio.loop(); 
}
/*// optional 
void audio_info(const char *info){
Serial.print("info "); Serial.println(info);
}
void audio_id3data(const char *info){ //id3 metadata
Serial.print("id3data ");Serial.println(info);
}
void audio_eof_mp3(const char *info){ //end of file
Serial.print("eof_mp3 ");Serial.println(info);
}
void audio_showstation(const char *info){
Serial.print("station ");Serial.println(info);
}
void audio_showstreaminfo(const char *info){
Serial.print("streaminfo ");Serial.println(info);
}
void audio_showstreamtitle(const char *info){
Serial.print("streamtitle ");Serial.println(info);
}
void audio_bitrate(const char *info){
Serial.print("bitrate ");Serial.println(info);
}
void audio_commercial(const char *info){ //duration in sec
Serial.print("commercial ");Serial.println(info);
}
void audio_icyurl(const char *info){ //homepage
Serial.print("icyurl ");Serial.println(info);
}
void audio_lasthost(const char *info){ //stream URL played
Serial.print("lasthost ");Serial.println(info);
}
void audio_eof_speech(const char *info){
Serial.print("eof_speech ");Serial.println(info);
}
*/
```
##### Funcionamiento
Ahora queremos reproducir un archivo WAVE. Por eso necesitamos una tarjeta SD con este archivo.
Para este proyecto necesitamos la librería Arduino ESP-audioI2S.
Primero, incorporamos todas las librerías y definimos los pins que utilizamos para conocer la ESP32 en el MAX98357A y en el módulo de la tarjeta SD.
A continuación inicializamos el objeto de audio con el nombre audio*.
En el void setup() definimos los pins y la conexión SPI por la comunicación de la tarjeta SD. También configuramos la velocidad de transmisión a 115200 bauds e inicializamos el objeto de la tarjeta SD.
Para el objeto de audio, los pins anteriores establecen el pin de salida. Reducimos el volumen del sonido a 10,
La última parte es conectar las entradas y salidas, es decir, el objeto de audio con el objeto de la tarjeta SD y definimos la ruta del archivo WAVE.
En el void loop() sólo hacemos un bucle sobre el objeto audio preconfigurado para reproducir la música.
La última parte sería por si queremos imprimir por pantalla algunos detalles de los archivos del sonido