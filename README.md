# MySQL-Data-Logging-for-CAN-sender

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

## Output

![image](https://github.com/KetanMe/MySQL-Data-Logging-for-CAN-sender/assets/121623546/92b0a40d-5b04-4af7-9f3b-fc508512ff8f)





