// This #include statement was automatically added by the Particle IDE.
#include <Adafruit_DHT.h>

#define DOOR_SENSOR_PIN     D0
#define WINDOW_SENSOR_PIN   D1
#define ON_BOARD_LED        D7
// #define FAN_LIGHT           A0
// #define FAN_OFF             D5
// #define FAN_HIGH            D4
// #define FAN_MEDIUM          D3
// #define FAN_LOW             D1
 #define FAN_LOW             D1

#define CLOSED  0
#define OPEN    1 
#define LOW     0
#define HIGH    1


/************************************************************/
/* DHT 22 Sensor section */
#define DHTPIN D2     // what pin we're connected to

#define DHTTYPE DHT11		// DHT 22 (AM2302)

// Connect pin 1 (on the left) of the sensor to +5V
// Connect pin 2 of the sensor to whatever your DHTPIN is
// Connect pin 4 (on the right) of the sensor to GROUND
// Connect a 10K resistor from pin 2 (data) to pin 1 (power) of the sensor
DHT dht(DHTPIN, DHTTYPE);

/* Global Variables */
char resultstr[64];

double humidity=0;    
double temperature=0; 
int DoorState = CLOSED;
int Prev_DoorState = 1;


/*****************************************************************************************************/
void setup() 
{
    /* Peripheral Configuration */
    Serial.begin(9600);                         // Serial Port Configuration
    pinMode(DOOR_SENSOR_PIN,INPUT_PULLUP);             // Door Sensor
    pinMode(WINDOW_SENSOR_PIN,INPUT_PULLDOWN);  // Window Sensor
    pinMode(ON_BOARD_LED,OUTPUT);               // System Armed Indicator
    // pinMode(FAN_LIGHT, OUTPUT); 
    // pinMode(FAN_HIGH, OUTPUT); 
    // pinMode(FAN_MEDIUM, OUTPUT); 
    pinMode(FAN_LOW, OUTPUT); 
    // pinMode(FAN_OFF, OUTPUT); 

    // digitalWrite(FAN_LIGHT, HIGH);
    // digitalWrite(FAN_OFF, HIGH);
    // digitalWrite(FAN_HIGH, HIGH);
    // digitalWrite(FAN_MEDIUM, HIGH);
    digitalWrite(FAN_LOW, HIGH);


    /* Particle Variables to publish */
    Particle.variable("Temperature",temperature); 
    Particle.variable("Humidity",humidity); 
    Particle.variable("Result", resultstr, STRING); 
    Particle.variable("Door State:", DoorState); 
   
    /* DHT Sensot setup */
	Serial.println("DHT11 test!");
	dht.begin();
	delay(2000);
}

 
/*****************************************************************************************************/
void loop() 
{
     
    
    /* Reading Temperature and Humidity Sensors */
	humidity = dht.getHumidity();                    // Read Humidity
	temperature = dht.getTempFarenheit()-10;  // Read temperature
	
	Serial.print("Humid: "); 
	Serial.print(humidity);
	Serial.print("% - ");
	Serial.print("Temp: "); 
	Serial.print(temperature);
	Serial.println("*F");

    /* Build a Json with the sensor information to make it available for G Docs */
    sprintf(resultstr, "{\"Temperature\":%2f,\"Humidity\":%2f}", temperature, humidity); 

    if(temperature > 78)
    {
        delay (3000);
        if(temperature > 78)
        {
      
        digitalWrite(FAN_LOW, LOW);
        delay(1000);
        // Particle.publish("Fan","ON"); 
        // digitalWrite(FAN_LOW, );
        // delay(300000); // 5 mins
        }
    }

    if(temperature  < 77)
    {
        delay (3000);
        if(temperature < 77)
        {
      
        digitalWrite(FAN_LOW, HIGH);
        delay(1000);
        // Particle.publish("Fan","OFF"); 
        // digitalWrite(FAN_LOW, );
        // delay(300000); // 5 mins
        }
    }

    /* Checking Door and Window Sensors */
    if(digitalRead(DOOR_SENSOR_PIN))
    {
        DoorState=1;
    }
    else
    {
        DoorState=0;
    }

    /* Checking last door status  */
    if(DoorState!=Prev_DoorState)
    {
        Prev_DoorState = DoorState;
        if(DoorState == OPEN)
        {
            Serial.println("Door Closed");
            #ifndef PUBLISH_DISABLED 
            Particle.publish("Door","Closed");
            #endif
        }
        else
        {
            Serial.println("Door Open");
            #ifndef PUBLISH_DISABLED
            Particle.publish("Door","Open");
            #endif
        }
    }
    delay(2000);
}