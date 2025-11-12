## ⚠️ Important Note on Confidentiality

This project is a backend vehicle scheduling system developed during my internship.

**Out of respect for the company's intellectual property and Non-Disclosure Agreement (NDA), this repository does not contain any source code.**

This README document serves as a detailed project overview, intended to demonstrate the features I was responsible for, the system architecture, the API design philosophy, and the technology stack used.

---

# Vehicle Scheduling System (VSS)

## Overview
The **Vehicle Scheduling System (VSS)** is a Java Spring Boot–based application designed to optimize and automate the enterprise's internal process of vehicle application, dispatching, cost-sharing, and management. This project digitizes the complete offline approval workflow, improves vehicle resource utilization, and enhances operational efficiency for company-wide transportation.

## Features
This system digitizes the entire vehicle request workflow for three main user roles:

* **👤 As an Applicant (Employee):**
    * Submit vehicle requests (Orders), specifying departure/return times, locations, and user ID (`/vss/orderForm/insertOrder`).
    * Submit a "Carpool" (Sharing Entry) request to join an existing approved order (`/vss/sharingEntry/insertSharingEntry`).
    * Cancel their carpool/sharing entry (`/vSss/sharingEntry/cancelEntry`).
    * Track the real-time status of their applications.

* **👨‍💼 As an Administrator (Dispatcher):**
    * Review, approve, or reject incoming vehicle applications.
    * **Manually dispatch** (update) an order to assign a specific `carId` and `driverId` (`/vss/orderForm/updateOrder`).
    * Manage the master lists of company drivers and vehicles (CRUD operations via `CarInfoController` and `DriverInfoController`).
    * Manually update a car's status (e.g., depart, arrive, free) (`/vss/carInfo/updateCarStatus`, `/vss/carInfo/resetCarStatus`).
    * Create a return-trip order based on an existing order (`/vss/orderForm/creatReturnOrder`).

* **⚙️ System-level Features:**
    * **Carpool/Sharing Logic:** Automatically checks if an existing order has available seats and is not yet departed before allowing a new user to join.
    * **Pagination:** All master lists (drivers, cars) support pagination for efficient display.
    * **Data Management:** Supports soft-delete (`deleteOrder`) and restore (`restoreOrder`) for order forms, ensuring data integrity.
    * **User Management:** Basic user creation and search capabilities (`/vss/userInfo/insertUser`).

## System Architecture
The system is developed using a layered architecture:
1.  **Controller Layer** — Handles HTTP requests and API routing (`com.vss.vehicleschedulingsystem.controller`).
2.  **Service Layer** — Contains business logic and scheduling algorithms (`com.vss.vehicleschedulingsystem.service`).
3.  **Mapper Layer** — Interfaces with the database using **MyBatis-Plus** (`com.vss.vehicleschedulingsystem.mapper`).
4.  **Domain Layer** — Defines domain entities/POJOs (`com.vss.vehicleschedulingsystem.domain`).

## Tech Stack
* **Java Version:** 1.8
* **Framework:** Spring Boot 2.6.13
* **Persistence:** MyBatis-Plus
* **Database:** MySQL
* **Database Pool:** Alibaba Druid
* **Pagination:** PageHelper
* **Build Tool:** Maven
* **Utilities:** Lombok

## API Endpoints
All endpoints are prefixed with `/vss`. The API accepts `application/x-www-form-urlencoded` (form data) parameters.

### `OrderFormController` (Core Logic)
* `POST /vss/orderForm/insertOrder`: Submits a new vehicle application.
    * *Params:* `OrderForm` object fields (e.g., `linkUserId`, `linkDepartPlace`, `linkDestination`, `linkDepartTime`).
* `POST /vss/orderForm/updateOrder`: Updates an existing order. (Used by Admin to dispatch/assign `linkCarId` and `linkDriverId`).
    * *Params:* `OrderForm` object fields (must include `orderId`).
* `GET /vss/orderForm/searchOrderById`: Retrieves a single order's details.
    * *Params:* `orderId` (as `@RequestParam`).
* `GET /vss/orderForm/deleteOrder`: Soft-deletes an order.
    * *Params:* `orderId` (as `@RequestParam`).
* `GET /vss/orderForm/restoreOrder`: Restores a soft-deleted order.
    * *Params:* `orderId` (as `@RequestParam`).

### `SharingEntryController` (Carpooling)
* `POST /vss/sharingEntry/insertSharingEntry`: Adds a passenger to an existing order (carpooling).
    * *Params:* `SharingEntry` object fields (e.g., `entryOrderId`, `entryUserId`).
* `POST /vss/sharingEntry/cancelEntry`: A passenger cancels their carpool entry.
    * *Params:* `orderId`, `userId`.

### `CarInfoController` (Admin)
* `POST /vss/carInfo/insertCar`: Adds a new vehicle to the fleet.
    * *Params:* `CarInfo` object fields.
* `POST /vss/carInfo/updateCar`: Updates vehicle details.
    * *Params:* `@RequestBody CarInfo` object. **(Note: This one uses JSON)**.
* `POST /vss/carInfo/updateCarStatus`: Manually updates a car's status (e.g., on a trip, available).
    * *Params:* `orderId`, `carId`.
* `GET /vss/carInfo/displayCarByPage`: Gets a paginated list of all cars.
    * *Params:* `pageNum`, `pageSize`.

### `DriverInfoController` (Admin)
* `POST /vss/driverInfo/insertDriver`: Adds a new driver.
    * *Params:* `DriverInfo` object fields.
* `POST /vss/driverInfo/updateDriver`: Updates a driver's details.
    * *Params:* `DriverInfo` object fields.
* `GET /vss/driverInfo/displayDriverByPage`: Gets a paginated list of all drivers.
    * *Params:* `pageNum`, `pageSize`.

### `UserInfoController` (Admin)
* `POST /vss/userInfo/insertUser`: Creates a new user in the system.
    * *Params:* `UserInfo` object fields.
* `POST /vss/userInfo/selectUserById`: Searches for a user.
    * *Params:* `UserInfo` object (uses `userId` from it).

## 🚀 Future Improvements
While the current version fulfills core business needs, the following improvements are planned to make the system more robust, efficient, and scalable:

* **1. Introduce Security Framework (Spring Security)**
    * **Current Status:** The API endpoints are open, lacking strict authentication and authorization.
    * **Improvement:** Integrate Spring Security + JWT (JSON Web Tokens) to provide Role-Based Access Control (RBAC). This would ensure, for example, that only users with an "Admin" role can call admin-related APIs, while "Drivers" and "Employees" are restricted to their respective endpoints.

* **2. Introduce Caching (Redis)**
    * **Current Status:** High-frequency, low-change queries (like fetching vehicle status or driver lists) directly hit the MySQL database.
    * **Improvement:** Implement Redis as a second-level cache. Caching this data would significantly reduce database load and improve system response times.

* **3. Optimize Logging (Logback/SLF4J)**
    * **Current Status:** The project currently relies on `System.out.println` for debugging, which is inefficient and difficult to manage in a production environment.
    * **Improvement:** Replace this with the standard SLF4J + Logback framework to manage logs. This allows for log rotation, different log levels (INFO, WARN, ERROR), and persistent log files for easier troubleshooting and monitoring.

* **4. Real-time Notifications (WebSocket / Message Queue)**
    * **Current Status:** Drivers and employees must manually refresh their clients to see new orders or status updates.
    * **Improvement:**
        * (Option A: WebSocket) Implement WebSockets to allow the server to "push" new task assignments directly to the driver's client in real-time.
        * (Option B: Message Queue) Integrate a message broker like RabbitMQ. Dispatching or approval events would be published as messages, allowing for asynchronous notifications and better system decoupling.

## Author
Developed by Haopeng Yu  
📧 peter.hp.yu@gmail.com
🌐 GitHub: https://github.com/peterhpyu