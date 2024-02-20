# MySQL-Data-Logging-for-CAN-sender

## Table of Contents
1. [Introduction](#introduction)
2. [MySQL: Basic Overview](#mysql-basic-overview)
3. [Block Diagram](#block-diagram)
4. [IoT Layers Used in the Project](#iot-layers-used-in-the-project)
5. [Protocols, Services, and Models](#protocols-services-and-models)
6. [Code Explanation](#code-explanation)
   - [Includes and Global Variables](#includes-and-global-variables)
   - [Setup Function](#setup-function)
   - [Loop Function](#loop-function)
   - [Helper Functions](#helper-functions)
7. [PHP Script for MySQL](#php-script-for-mysql)
   - [Explanation of PHP Script](#explanation-of-php-script)
8. [Output](#output)

## Introduction
In this project I have used MySQL database to log the data of e-ATV parameters.This project will send the data on 

## MySQL: Basic Overview


1. **Relational Database**: MySQL follows the relational model, which means it organizes data into tables consisting of rows and columns. Each table represents an entity, and relationships between entities are established using keys.

2. **Structured Query Language (SQL)**: MySQL uses SQL as its primary interface for interacting with the database. SQL allows users to perform various operations such as creating, reading, updating, and deleting data (CRUD operations), as well as defining the structure of the database (DDL operations) and managing access permissions.

3. **Tables**: In MySQL, data is stored in tables, which are organized into rows and columns. Each column has a data type that defines the kind of data it can store (e.g., integer, string, date), and each row represents a record or data entry.

4. **Primary Keys and Foreign Keys**: Primary keys are unique identifiers for rows in a table, ensuring that each row can be uniquely identified. Foreign keys establish relationships between tables by referencing the primary key of another table.

5. **Indexes**: Indexes are data structures that improve the performance of database queries by allowing faster retrieval of data. They are created on one or more columns of a table and help speed up search operations.

6. **Transactions**: MySQL supports transactions, which are sequences of operations that are treated as a single unit of work. Transactions ensure data integrity and consistency by allowing users to perform multiple operations as a single atomic operation.

7. **User Management and Access Control**: MySQL provides mechanisms for user authentication and access control, allowing administrators to grant or restrict access to databases and tables based on user roles and privileges.

8. **Scalability and High Availability**: MySQL is designed to scale from small applications to large-scale enterprise systems. It supports features like replication, clustering, and sharding to achieve high availability and scalability.

9. **Community and Support**: MySQL has a large and active community of users and developers who contribute to its development, provide support, and share resources such as tutorials, forums, and documentation.

## Block Diagram
![Block Diagram (1)](https://github.com/KetanMe/MySQL-Data-Logging-for-CAN-sender/assets/121623546/62277c7f-8e5b-431d-a143-f705b641788e)

## IoT Layers Used in the project 

| Layer            | Description                |
|------------------|----------------------------|
| Application      | Application Layer          |
| MySQL Server     | Database Management System |
| Middleware       | Intermediate Processing    |
| Communication    | Data Transmission Layer    |
| Device           | Physical Devices           |


**Device Layer**: This layer includes physical devices such as sensors, actuators, and microcontrollers (e.g., ESP8266). In this project, the DS18B20 temperature sensor and the M274 rotary encoder are part of this layer.

**Communication Layer**: This layer is responsible for establishing communication between devices and the higher layers of the IoT architecture. It includes protocols and technologies such as Wi-Fi, Ethernet, Bluetooth, Zigbee, etc. In this project, the ESP8266 communicates with the MySQL server over Wi-Fi or Ethernet.

**Middleware Layer**: The middleware layer acts as an intermediary between the device layer and the application layer. It handles tasks such as data preprocessing, protocol translation, and message routing. In this project, the ESP8266 may perform some middleware functions to process the data before sending it to the MySQL server.

**Application Layer**: This is the top layer of the IoT architecture, where applications and services are developed to analyze, visualize, and act upon the data collected from the devices. In this project, the MySQL server stores the data, and applications or services can access this data for analysis and visualization.


Each layer interacts with the layers above and below it, enabling data flow from the device layer to the application layer and vice versa. This layered architecture provides a scalable and modular framework for building IoT systems.

## Protocols, Services and Models

**IoT Protocol**: 

![image](https://github.com/KetanMe/MySQL-Data-Logging-for-CAN-sender/assets/121623546/efbaaa23-efbb-448e-a982-3603330d6619)


   - For communication between the ESP8266 microcontroller and the MySQL server, the project likely uses the HTTP or HTTPS protocol over Wi-Fi. HTTP (Hypertext Transfer Protocol) is a widely used protocol for communication on the web, while HTTPS (HTTP Secure) adds a layer of encryption using SSL/TLS to secure the data transmission. These protocols are commonly used in IoT applications for sending data to remote servers.

**IoT Service**:
   - The MySQL server acts as the IoT service in this project. It provides a database management system that stores the data sent by the ESP8266 microcontroller. MySQL allows for the efficient storage, retrieval, and management of structured data, making it suitable for IoT applications where sensor data needs to be logged and analyzed.

**Model**:

![image](https://github.com/KetanMe/MySQL-Data-Logging-for-CAN-sender/assets/121623546/a27b9211-e539-4349-9b30-ba9f66735023)

   - The project follows the Client-Server model. In this model, the ESP8266 microcontroller acts as the client, responsible for collecting data from the sensors (DS18B20 temperature sensor and M274 rotary encoder) and sending it to the MySQL server. The MySQL server acts as the server, receiving the data from the client and storing it in the database. This client-server architecture enables the separation of concerns between data collection (client) and data storage/processing (server), allowing for scalability and flexibility in IoT applications.

## Code 
```cpp
#include <Wire.h>
#include <mcp2515.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

// WiFi settings
const char *ssid = "Galaxy";
const char *password = "12345678";

// Server URL for logging data
const char *serverURL = "http://192.168.245.231/ds18b20_testing/test_data.php";

// WiFi client
WiFiClient client;

// CAN bus settings
#define SENDER_CAN_ID 0x124
const int spiCS = D8;
MCP2515 mcp2515Sender(spiCS);

// Rotary encoder pins
#define CLK D0
#define DT D1

// Constants for RPM calculation
const int CPR = 20;
const float radius = 0.03;

// Temperature sensor pins
#define SENSOR_1_BUS D2
#define SENSOR_2_BUS D3
#define SENSOR_3_BUS D4

// Variables for storing temperature and RPM values
float temp1, temp2, temp3;
float rpm = 0;

// Rotary encoder variables
int counter = 0;
int currentStateCLK;
int lastStateCLK;

// Dallas temperature sensors
OneWire sensor1Wire(SENSOR_1_BUS);
OneWire sensor2Wire(SENSOR_2_BUS);
OneWire sensor3Wire(SENSOR_3_BUS);
DallasTemperature sensor1(&sensor1Wire);
DallasTemperature sensor2(&sensor2Wire);
DallasTemperature sensor3(&sensor3Wire);

// Enumeration for state machine
enum State {
  WAIT_FOR_DATA,
  PRINT_RPM_SPEED,
  PRINT_TEMP_1,
  PRINT_TEMP_2,
  PRINT_TEMP_3
};

// Initial state
State currentState = WAIT_FOR_DATA;

// Last update times for RPM and temperature readings
unsigned long lastMillisRPM = 0;
unsigned long lastMillisTemp = 0;

// Last temperature print time
unsigned long lastTempPrintMillis = 0;

// Function prototypes
void connectWiFi();
float readTemperature(DallasTemperature &sensor);
void logDataToServer(float temp1, float temp2, float temp3, float rpm);

void setup() {
  Serial.begin(9600);

  Serial.println("Initializing MCP2515 Sender...");
  mcp2515Sender.reset();
  if (mcp2515Sender.setBitrate(CAN_500KBPS, MCP_8MHZ) != MCP2515::ERROR_OK) {
    Serial.println("Error setting bitrate!");
    while (1);
  }
  mcp2515Sender.setNormalMode();
  Serial.println("MCP2515 Sender Initialized Successfully!");

  connectWiFi();
}

void loop() {
  unsigned long currentMillis = millis();
  currentStateCLK = digitalRead(CLK);

  if (currentStateCLK != lastStateCLK) {
    if (digitalRead(DT) != currentStateCLK) {
      counter--;
    } else {
      counter++;
    }
  }

  switch (currentState) {
    case WAIT_FOR_DATA:
      if (currentMillis - lastMillisRPM >= 1000) {
        currentState = PRINT_RPM_SPEED;
        lastMillisRPM = currentMillis;
      } else if (currentMillis - lastMillisTemp >= 10000) {
        currentState = PRINT_TEMP_1;
        lastMillisTemp = currentMillis;
      }
      break;

    case PRINT_RPM_SPEED:
      rpm = abs(counter * 60.0 / CPR);
      float speed = (2 * PI * radius * rpm) / 60.0;

      // Construct CAN message
      struct can_frame canMsg;
      canMsg.can_id = SENDER_CAN_ID;
      canMsg.can_dlc = 8;
      memcpy(canMsg.data, &rpm, sizeof(rpm));
      memcpy(canMsg.data + sizeof(rpm), &speed, sizeof(speed));

      // Send CAN message
      mcp2515Sender.sendMessage(&canMsg);

      // Log data to server
      logDataToServer(temp1, temp2, temp3, rpm);

      counter = 0;
      currentState = WAIT_FOR_DATA;
      break;

    case PRINT_TEMP_1:
    case PRINT_TEMP_2:
    case PRINT_TEMP_3:
      if (currentMillis - lastTempPrintMillis >= 1000) {
        // Read temperature from the corresponding sensor
        float temp = readTemperature(currentState == PRINT_TEMP_1 ? sensor1 : (currentState == PRINT_TEMP_2 ? sensor2 : sensor3));

        // Construct CAN message
        struct can_frame canMsg;
        canMsg.can_id = SENDER_CAN_ID;
        canMsg.can_dlc = 4;
        memcpy(canMsg.data, &temp, sizeof(temp));

        // Send CAN message
        mcp2515Sender.sendMessage(&canMsg);

        // Log data to server
        logDataToServer(temp1, temp2, temp3, rpm);

        lastTempPrintMillis = currentMillis;

        // Update temperature variables based on state
        if (currentState == PRINT_TEMP_1) {
          temp1 = temp;
          currentState = PRINT_TEMP_2;
        } else if (currentState == PRINT_TEMP_2) {
          temp2 = temp;
          currentState = PRINT_TEMP_3;
        } else {
          temp3 = temp;
          currentState = WAIT_FOR_DATA;
        }
      }
      break;
  }

  lastStateCLK = currentStateCLK;
}

// Function to read temperature from a DallasTemperature sensor
float readTemperature(DallasTemperature &sensor) {
  sensor.requestTemperatures();
  float tempC = sensor.getTempCByIndex(0);

  Serial.print("Temperature: ");
  Serial.println(tempC);

  return tempC;
}

// Function to log data to the server
void logDataToServer(float temp1, float temp2, float temp3, float rpm) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    http.begin(client, serverURL);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    String postData = "temp1=" + String(temp1) + "&temp2=" + String(temp2) + "&temp3=" + String(temp3) + "&rpm=" + String(rpm);

    int httpCode = http.POST(postData);

    Serial.print("Server URL: ");
    Serial.println(serverURL);
    Serial.print("Data sent to server: ");
    Serial.println(postData);
    Serial.print("HTTP Code: ");
    Serial.println(httpCode);

    if (httpCode == HTTP_CODE_OK) {
      String payload = http.getString();
      Serial.print("Server Response: ");
      Serial.println(payload);
    } else {
      Serial.println("Failed to send data to server.");
      Serial.println(http.errorToString(httpCode).c_str());
    }

    http.end();
  } else {
    Serial.println("WiFi not connected. Unable to send data to server.");
  }
}

// Function to connect to WiFi
void connectWiFi() {
  WiFi.disconnect();
  delay(1000);

  WiFi.begin(ssid, password);
  Serial.println("Connecting to WiFi");

  int attemptCounter = 0;

  while (WiFi.status() != WL_CONNECTED && attemptCounter < 30) {
    delay(500);
    Serial.print(".");
    attemptCounter++;
 

 }

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("\nFailed to connect to WiFi. Please check your credentials.");
  } else {
    Serial.println("\nConnected to WiFi");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
  }
}
```

## Explaination of code 

Sure, let's break down the code part by part:

1. **Includes and Global Variables**:
    - The code includes necessary libraries for CAN communication, Dallas Temperature sensor, WiFi, and HTTP client.
    - It defines global variables including WiFi credentials, server URL, CAN bus settings, pin configurations for sensors and rotary encoder, temperature and RPM values, and state variables for the state machine.

```cpp
#include <Wire.h>
#include <mcp2515.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

const char *ssid = "Galaxy";
const char *password = "12345678";

const char *serverURL = "http://192.168.245.231/ds18b20_testing/test_data.php";

WiFiClient client;

#define SENDER_CAN_ID 0x124
const int spiCS = D8;
MCP2515 mcp2515Sender(spiCS);

#define CLK D0
#define DT D1

const int CPR = 20;
const float radius = 0.03;

#define SENSOR_1_BUS D2
#define SENSOR_2_BUS D3
#define SENSOR_3_BUS D4

float temp1, temp2, temp3;
float rpm = 0;

int counter = 0;
int currentStateCLK;
int lastStateCLK;

OneWire sensor1Wire(SENSOR_1_BUS);
OneWire sensor2Wire(SENSOR_2_BUS);
OneWire sensor3Wire(SENSOR_3_BUS);
DallasTemperature sensor1(&sensor1Wire);
DallasTemperature sensor2(&sensor2Wire);
DallasTemperature sensor3(&sensor3Wire);

enum State {
  WAIT_FOR_DATA,
  PRINT_RPM_SPEED,
  PRINT_TEMP_1,
  PRINT_TEMP_2,
  PRINT_TEMP_3
};

State currentState = WAIT_FOR_DATA;

unsigned long lastMillisRPM = 0;
unsigned long lastMillisTemp = 0;
unsigned long lastTempPrintMillis = 0;
```

2. **Setup Function**:
    - Initializes serial communication for debugging.
    - Initializes MCP2515 CAN controller.
    - Connects to WiFi network.
    - Initializes the state machine.

```cpp
void setup() {
  Serial.begin(9600);

  Serial.println("Initializing MCP2515 Sender...");
  mcp2515Sender.reset();
  if (mcp2515Sender.setBitrate(CAN_500KBPS, MCP_8MHZ) != MCP2515::ERROR_OK) {
    Serial.println("Error setting bitrate!");
    while (1);
  }
  mcp2515Sender.setNormalMode();
  Serial.println("MCP2515 Sender Initialized Successfully!");

  connectWiFi();
}
```

3. **Loop Function**:
    - Reads rotary encoder values and calculates RPM.
    - Based on the state machine, either sends RPM and speed data or reads temperature data from sensors and sends it.
    - Logs data to the server.

```cpp
void loop() {
  unsigned long currentMillis = millis();
  currentStateCLK = digitalRead(CLK);

  // Rotary encoder reading
  if (currentStateCLK != lastStateCLK) {
    if (digitalRead(DT) != currentStateCLK) {
      counter--;
    } else {
      counter++;
    }
  }

  // State machine logic
  switch (currentState) {
    case WAIT_FOR_DATA:
      if (currentMillis - lastMillisRPM >= 1000) {
        currentState = PRINT_RPM_SPEED;
        lastMillisRPM = currentMillis;
      } else if (currentMillis - lastMillisTemp >= 10000) {
        currentState = PRINT_TEMP_1;
        lastMillisTemp = currentMillis;
      }
      break;

    case PRINT_RPM_SPEED:
      // RPM and speed calculation
      rpm = abs(counter * 60.0 / CPR);
      float speed = (2 * PI * radius * rpm) / 60.0;

      // Construct CAN message
      struct can_frame canMsg;
      canMsg.can_id = SENDER_CAN_ID;
      canMsg.can_dlc = 8;
      memcpy(canMsg.data, &rpm, sizeof(rpm));
      memcpy(canMsg.data + sizeof(rpm), &speed, sizeof(speed));

      // Send CAN message
      mcp2515Sender.sendMessage(&canMsg);

      // Log data to server
      logDataToServer(temp1, temp2, temp3, rpm);

      counter = 0;
      currentState = WAIT_FOR_DATA;
      break;

    case PRINT_TEMP_1:
    case PRINT_TEMP_2:
    case PRINT_TEMP_3:
      if (currentMillis - lastTempPrintMillis >= 1000) {
        // Read temperature from the corresponding sensor
        float temp = readTemperature(currentState == PRINT_TEMP_1 ? sensor1 : (currentState == PRINT_TEMP_2 ? sensor2 : sensor3));

        // Construct CAN message
        struct can_frame canMsg;
        canMsg.can_id = SENDER_CAN_ID;
        canMsg.can_dlc = 4;
        memcpy(canMsg.data, &temp, sizeof(temp));

        // Send CAN message
        mcp2515Sender.sendMessage(&canMsg);

        // Log data to server
        logDataToServer(temp1, temp2, temp3, rpm);

        lastTempPrintMillis = currentMillis;

        // Update temperature variables based on state
        if (currentState == PRINT_TEMP_1) {
          temp1 = temp;
          currentState = PRINT_TEMP_2;
        } else if (currentState == PRINT_TEMP_2) {
          temp2 = temp;
          currentState = PRINT_TEMP_3;
        } else {
          temp3 = temp;
          currentState = WAIT_FOR_DATA;
        }
      }
      break;
  }

  lastStateCLK = currentStateCLK;
}
```

4. **Helper Functions**:
    - `readTemperature`: Reads temperature from a DallasTemperature sensor.
    - `logDataToServer`: Logs temperature and RPM data to the server.
    - `connectWiFi`: Connects to WiFi network.

```cpp
float readTemperature(DallasTemperature &sensor) {
  sensor.requestTemperatures();
  float tempC = sensor.getTempCByIndex(0);

  Serial.print("Temperature: ");
  Serial.println(tempC);

  return tempC;
}

void logDataToServer(float temp1, float temp2, float temp3, float rpm) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    http.begin(client, serverURL);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    String postData = "temp1=" + String(temp1) + "&temp2=" + String(temp2) + "&temp3=" + String(temp3) + "&rpm=" + String(rpm);

    int httpCode = http.POST(postData);

    Serial.print("Server URL: ");
    Serial.println(serverURL);
    Serial.print("Data sent to server: ");
    Serial.println(postData);
    Serial.print("HTTP Code: ");
    Serial.println(httpCode);

    if (httpCode == HTTP_CODE_OK) {
      String payload = http.getString();
      Serial.print("Server Response: ");
      Serial.println(payload);
    } else {
      Serial.println("Failed to send data to server.");
      Serial.println(http.errorToString(httpCode).c_str());
    }

    http.end();
  } else

 {
    Serial.println("WiFi not connected. Unable to send data to server.");
  }
}

void connectWiFi() {
  WiFi.disconnect();
  delay(1000);

  WiFi.begin(ssid, password);
  Serial.println("Connecting to WiFi");

  int attemptCounter = 0;

  while (WiFi.status() != WL_CONNECTED && attemptCounter < 30) {
    delay(500);
    Serial.print(".");
    attemptCounter++;
  }

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("\nFailed to connect to WiFi. Please check your credentials.");
  } else {
    Serial.println("\nConnected to WiFi");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
  }
}
```

This code is designed to interface with a CAN bus, read RPM from a rotary encoder, read temperatures from Dallas Temperature sensors, and log data to a server over WiFi. It utilizes a state machine to manage different tasks efficiently and follows best practices for IoT device development.

## PHP script for MySQL

```php
<?php
$hostname = "localhost";  // Replace with your actual MySQL server IP or hostname
$username = "root";
$password = "";
$database = "testing";

$conn = mysqli_connect($hostname, $username, $password, $database);

if (!$conn) {
    die("Connection fail" . mysqli_connect_error());
}

// Check if temp1, temp2, and temp3 are set in the POST request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if (isset($_POST["temp1"]) && isset($_POST["temp2"]) && isset($_POST["temp3"])) {
        $t1 = $_POST["temp1"];
        $t2 = $_POST["temp2"];
        $t3 = $_POST["temp3"];

        // Prepare and execute the SQL query
        $sql = "INSERT INTO table_1 (temp1, temp2, temp3) VALUES (" . $t1 . ", " . $t2 . ", " . $t3 . ")";

        if (mysqli_query($conn, $sql)) {
            echo "\nNew record created";
        } else {
            echo "Error: " . $sql . "<br>" . mysqli_error($conn);
        }
    } else {
        echo "Invalid data received. temp1, temp2, and temp3 are required.";
    }
} else {
    echo "Invalid request method. Only POST requests are allowed.";
}

mysqli_close($conn);
?>
```

## Explaination of PHP script

1. **Database Connection**: The script starts by defining variables for the database connection parameters - `$hostname`, `$username`, `$password`, and `$database`. Then, it attempts to establish a connection to the MySQL database using the `mysqli_connect()` function. If the connection fails, it terminates the script and outputs an error message.

2. **Handling POST Requests**: The script checks if the request method is POST using `$_SERVER["REQUEST_METHOD"]`. If it is a POST request, it proceeds to check if `temp1`, `temp2`, and `temp3` parameters are set in the POST data using `isset()`.

3. **Inserting Data into the Database**: If all three parameters are set, their values are retrieved from the POST data (`$_POST`) and stored in variables `$t1`, `$t2`, and `$t3`. Then, an SQL query is constructed to insert these values into a table named `table_1` in the database. The query is executed using `mysqli_query()`. If the query is successful, it outputs a message indicating that a new record has been created. If there is an error executing the query, it outputs an error message along with the SQL query and the specific error returned by MySQL.

4. **Error Handling for Invalid Data**: If `temp1`, `temp2`, or `temp3` parameters are missing in the POST data, it outputs a message indicating that invalid data was received and that these parameters are required.

5. **Error Handling for Invalid Request Method**: If the request method is not POST, it outputs a message indicating that only POST requests are allowed.

6. **Closing Database Connection**: Finally, the script closes the database connection using `mysqli_close()`.

## Output

![image](https://github.com/KetanMe/MySQL-Data-Logging-for-CAN-sender/assets/121623546/92b0a40d-5b04-4af7-9f3b-fc508512ff8f)

See the .csv file [here](https://github.com/KetanMe/MySQL-Data-Logging-for-CAN-sender/blob/main/table_1.csv)




