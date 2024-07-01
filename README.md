# Proyecto de Robótica
Integrantes:
- Aldo Jiménez
- Jorge Figueroa

## Explicación del código
### Definición de variables 

En esta parte del código estamos definiendo variables las cuales van a ser usadas más adelante, principalmente son los pines a los cuales conectamos motores y sensores.

```cpp
#define MOTOR1_IN2 5
#define MOTOR2_ENABLE 10
#define MOTOR2_IN1 8
#define MOTOR2_IN2 9

QTRSensors qtr;
bool isComment = 0;
const uint8_t SensorCount = 6;
uint16_t sensorValues[SensorCount];
```

### Funciones 

En la siguiente sección del código tenemos la creación de las funciones que nos permitirán movernos en todas las direcciones.

```cpp
void turnright(int x, int y){
   digitalWrite(MOTOR1_IN1, HIGH);
    digitalWrite(MOTOR1_IN2, LOW);
    digitalWrite(MOTOR2_IN1, LOW);
    digitalWrite(MOTOR2_IN2, HIGH);
    analogWrite(MOTOR1_ENABLE, x);
    analogWrite(MOTOR2_ENABLE, y);
}
void turnleft(int x, int y){
  digitalWrite(MOTOR1_IN1, LOW);
    digitalWrite(MOTOR1_IN2, HIGH);
    digitalWrite(MOTOR2_IN1, HIGH);
    digitalWrite(MOTOR2_IN2, LOW);
    analogWrite(MOTOR1_ENABLE, x);
    analogWrite(MOTOR2_ENABLE, y);
}
void forward(int vel){
  digitalWrite(MOTOR1_IN1, LOW);
    digitalWrite(MOTOR1_IN2, HIGH);
    digitalWrite(MOTOR2_IN1, LOW);
    digitalWrite(MOTOR2_IN2, HIGH);
    analogWrite(MOTOR1_ENABLE, vel);
    analogWrite(MOTOR2_ENABLE, vel);
}
void stops(){
  digitalWrite(MOTOR1_IN1, LOW);
    digitalWrite(MOTOR1_IN2, LOW);
    digitalWrite(MOTOR2_IN1, LOW);
    digitalWrite(MOTOR2_IN2, LOW);
    analogWrite(MOTOR1_ENABLE, 0);
    analogWrite(MOTOR2_ENABLE, 0);
}
```

### Void Setup

Dentro de void setup, el cual es la parte del código que se corre primero al encender el Arduino, tenemos en primer lugar la definición de los pines y cómo estos se van a comportar. Además de configurar los sensores que se encuentran en la parte de abajo de nuestro robot, luego de medio milisegundo empieza la calibración del robot donde este empieza a dar vueltas en círculos analizando los sensores y así calibrándose para distinguir el negro del blanco. Al finalizar esto toma una pausa de un segundo.

```cpp
void setup() {
   // configure the sensors
  pinMode(MOTOR1_ENABLE, OUTPUT);
  pinMode(MOTOR1_IN1, OUTPUT);
  pinMode(MOTOR1_IN2, OUTPUT);
  pinMode(MOTOR2_ENABLE, OUTPUT);
  pinMode(MOTOR2_IN1, OUTPUT);
  pinMode(MOTOR2_IN2, OUTPUT);
  qtr.setTypeAnalog();
  qtr.setSensorPins((const uint8_t[]){A0, A1, A2, A3, A4, A5}, SensorCount);

  delay(500);
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, HIGH); // turn on Arduino's LED to indicate we are in calibration mode
   turnleft(100,100);
  // analogRead() takes about 0.1 ms on an AVR.
  // 0.1 ms per sensor * 4 samples per sensor read (default) * 6 sensors
  // * 10 reads per calibrate() call = ~24 ms per calibrate() call.
  // Call calibrate() 400 times to make calibration take about 10 seconds.
  for (uint16_t i = 0; i < 400; i++)
  {
    qtr.calibrate();
  }
  digitalWrite(LED_BUILTIN, LOW); // turn off Arduino's LED to indicate we are through with calibration

  // print the calibration minimum values measured when emitters were on
  Serial.begin(9600);
  for (uint8_t i = 0; i < SensorCount; i++)
  {
    Serial.print(qtr.calibrationOn.minimum[i]);
    Serial.print(' ');
  }
  Serial.println();

  // print the calibration maximum values measured when emitters were on
  for (uint8_t i = 0; i < SensorCount; i++)
  {
    Serial.print(qtr.calibrationOn.maximum[i]);
    Serial.print(' ');
  }
  Serial.println();
  Serial.println();
  delay(1000);// put your setup code here, to run once:

}
```

### Void Loop 

Finalmente, el void loop, el cual se repite constantemente para el funcionamiento de nuestro robot. Este código lo que hace, es en un comienzo guardar en tres variables los valores de nuestros 6 sensores, siendo estos el promedio de 2 sensores juntos. Luego iniciamos 4 variables las cuales van a ser 3 en caso de que el sensor detecte que hay negro y el otro va a ser el nivel de sensibilidad de la detección del color, es decir, el umbral de sensibilidad. Luego hacemos la detección del color y modificamos las variables de detección de color en caso de detectarse un cambio. A continuación tenemos una serie de ifs los cuales nos darán la estrategia a seguir, la cual será explicada con imágenes.

```cpp 
void loop() {
    // read calibrated sensor values and obtain a measure of the line position
  // from 0 to 5000 (for a white line, use readLineWhite() instead)
  uint16_t position = qtr.readLineBlack(sensorValues);
int sensorcentral= (sensorValues[2] + sensorValues[3])/2;
// int sensorleft= (sensorValues[0]+sensorValues[1])/2;
// int sensorright= (sensorValues[4]+sensorValues[5])/2;
int sensorright= (sensorValues[0]+sensorValues[1])/2;
int sensorleft= (sensorValues[4]+sensorValues[5])/2;

  // print the sensor values as numbers from 0 to 1000, where 0 means maximum
  // reflectance and 1000 means minimum reflectance, followed by the line
  // position

int threshold = 500;
bool isScBlack = 0;
bool isSiBlack = 0;
bool isSdBlack = 0;



if  (sensorcentral > threshold) {

    isScBlack =1;
} else {
  isScBlack=0;
}

if  (sensorleft > threshold) {

    isSiBlack =1;
} else {
  isSiBlack=0;
}

if  (sensorright > threshold) {
    isSdBlack =1;
} else {
  isSdBlack=0;
}

if (isScBlack){
  if(!isSiBlack & !isSdBlack){
    forward(150);
    if (!isComment)
    Serial.println("central negro y el resto blanco");
  }
  else if (!isSiBlack & isSdBlack){
    turnright(150,150);
    if (!isComment)
    Serial.println("central, derecho negro y el izquierda blanco");
  }
  else if (isSiBlack & !isSdBlack){
    turnleft(150,150);
    if (!isComment)
    Serial.println("central, izquiedo negro y el derecha blanco");
  }
  else if (isSiBlack & isSdBlack){
    turnleft(100,100);
    delay(500);
    Serial.print("todo negro");
  }
} 
else {
    if (isSiBlack & !isSdBlack){
      turnleft(150,150);
      if (!isComment)
      Serial.println("central, derecha blanco y izquieda negro");
    }
    else if (!isSiBlack & isSdBlack){
      turnright(150,150);
      if (!isComment)
      Serial.println("central, izquierda blanco y derecha negro");
    }
    else if (!isSiBlack & !isSdBlack){
      stops(); 
      if (!isComment)
      Serial.println("todo blanco");
    }

}






  if (!isComment){  
  for (uint8_t i = 0; i < SensorCount; i++)
  {
    Serial.print(sensorValues[i]);
    Serial.print('\t');
  }
  Serial.println(position);



  }
}
```
En primer lugar, indicamos en caso de ser negro el centro o no, ahí separamos en 2 situaciones. En caso de ser positivo se toma el primer caso que sería centro negro y resto blanco, aquí se toma la estrategia de ir adelante. En caso de ser negro al centro y a la derecha, y blanco a la izquierda, tomará la decisión de ir a la izquierda. Si es caso contrario, es decir, negro a la izquierda y blanco a la derecha, tomará la elección de ir a la derecha. Por último, en caso de ser todo negro, girará a la izquierda en círculos hasta encontrar un camino y al ser todo blanco se quedará quieto.

####  Hacia adelante 
 
Las flechas indican la direccion de las ruedas.

![hacia adelante](/img/hacia%20adelante.png)


####  Hacia la izquierda
 

![hacia la izquierda](/img/a%20la%20izquierda.png)

####  Hacia la derecha 
 

![hacia la derecha](/img/a%20la%20derecha.png)
####  Quieto 
 

![quieto](/img/stop.png)