# NSK - Full-Stack Multi-Tenant Booking System (THIS README FILE IS STILL IN PROGRESS)

![Project Status: Live](https://img.shields.io/badge/status-live%20%26%20in%20production-brightgreen)


---

> **Note:** This repository serves as a public showcase for the project. Due to its proprietary nature and commercial use, the source code is maintained in a private repository. This README provides a detailed overview of the project's architecture, features, and technical implementation.


---


## Table of Contents
* [About The Project](#about-the-project)
* [Key Features](#key-features)
* [Technical Architecture & Highlights](#technical-architecture--highlights)
  * [Race Condition & Concurrency Management](#race-condition--concurrency-management)
  * [Data Integrity with Atomic Transactions](#data-integrity-with-atomic-transactions)
  * [Scalable Multi-Tenant Design](#scalable-multi-tenant-design)
  * [Secure Authentication Flow](#secure-authentication-flow)
  * [Production Deployment & CI/CD](#production-deployment--cicd)
* [Technology Stack](#technology-stack)
* [Contact](#contact)

## About The Project

NSK is a production-grade, full-stack booking platform built from the ground up to manage sports facilities for multiple locations. Designed to handle significant traffic and revenue, the system provides a seamless experience for users while offering powerful administrative tools for staff. The platform is projected to manage over **₹4 lakh in monthly revenue**, highlighting its robustness and commercial viability.

## Key Features

- **Multi-Tenant Architecture:** Securely serves multiple independent sports clubs/locations from a single application instance, with complete data isolation.
- **Role-Based Access Control (RBAC):** Granular permissions for Super Admins, Location Admins, and Staff, ensuring users only access relevant data and functionality.
- **Real-Time Slot Booking:** Users can view available slots and book them in real-time without conflicts.
- **Race-Condition Proof:** A sophisticated system prevents double-bookings even under high concurrency.
- **Secure OTP Authentication:** Passwordless, mobile number-based login flow for a smooth and secure user experience.
- **Automated Deployments:** A complete CI/CD pipeline ensures rapid and reliable updates to the production environment.
- **Admin & Staff Dashboards:** Dedicated interfaces for managing courts, staff, bookings, and viewing location-specific analytics.

## Technical Architecture & Highlights

This section details the key engineering solutions that power the NSK platform.

### Race Condition & Concurrency Management

To guarantee **100% booking integrity** and eliminate the risk of double-bookings, a multi-layered solution was engineered:

1.  **Temporary Reservation Lock:** When a user selects a slot, a temporary reservation is created in the database with a 5-minute TTL (Time-To-Live). This slot is immediately marked as unavailable to other users.
2.  **Real-Time Disconnect Detection:** **WebSockets** are used to maintain a persistent connection with the client during the booking process. If the user closes the tab or loses connection, the server detects the disconnect instantly and releases the temporary lock, making the slot available again.
3.  **Scheduled Job Fail-Safe:** As a final layer of defense, a scheduled job runs using **Agenda.js** to periodically scan for and clear any expired temporary reservations that may not have been cleared by other mechanisms, ensuring no slot remains locked indefinitely.

### Data Integrity with Atomic Transactions

For features like bulk bookings where a user reserves multiple slots simultaneously, data consistency is critical. The entire operation must succeed or fail as a single unit. This is achieved using **MongoDB Sessions**, which wrap all related database operations (creating multiple booking records, updating user profiles, etc.) into a single **atomic transaction**. If any step fails, the entire transaction is rolled back, leaving the database in a consistent state.

### Scalable Multi-Tenant Design

The application is built on a multi-tenant architecture to serve different business locations from a single codebase and database.
- **Data Isolation:** Every relevant document in the database (e.g., `bookings`, `courts`, `users`) includes a `locationId` field. All database queries are scoped to the authenticated admin's specific `locationId`, ensuring they can only access and manage their own data.
- **Centralized Super Admin:** A super admin role exists outside this structure, with the ability to manage locations and oversee the entire system.

### Secure Authentication Flow

Security was a top priority. The authentication flow is designed to be both user-friendly and highly secure:
1.  **OTP Verification:** Users register and log in using their phone number, receiving an OTP via the **MSG91** service.
2.  **JWT Generation:** Upon successful OTP verification, a secure **JSON Web Token (JWT)** is generated.
3.  **HttpOnly Cookie Storage:** The JWT is stored in an **HttpOnly, SameSite=None, Secure cookie**. This is the critical step that prevents Cross-Site Scripting (XSS) attacks, as client-side JavaScript cannot access the token.

### Production Deployment & CI/CD

The entire deployment lifecycle is automated for efficiency and reliability:
- **Hosting:** The Node.js backend is deployed on an **AWS EC2** instance.
- **Reverse Proxy:** **Nginx** is configured as a reverse proxy to manage incoming traffic. It is responsible for **SSL termination** (enabling HTTPS) and proxying requests to the correct port for the Node.js application. It also handles the secure WebSocket protocol (`wss://`).
- **Automation:** A **CI/CD pipeline** is set up using **GitHub Actions**. On every `git push` to the main branch of the private repository, the pipeline automatically:
    1.  Connects to the EC2 instance via SSH.
    2.  Pulls the latest code.
    3.  Installs dependencies (`npm install`).
    4.  Restarts the application gracefully using **PM2**, ensuring zero downtime.

## Technology Stack

| Category              | Technologies                                                                                             |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| **Frontend** | `React`, `React Router`, `Axios`, `Tailwind CSS`, `Socket.IO Client`                                       |
| **Backend** | `Node.js`, `Express.js`, `Mongoose`, `Socket.IO`, `Agenda.js`                                              |
| **Database** | `MongoDB`                                                                                                |
| **DevOps & Deployment** | `AWS EC2`, `Nginx`, `PM2`, `GitHub Actions (CI/CD)`, `SSL (Let's Encrypt)`                                 |
| **APIs & Services** | `MSG91` (for OTP), `JWT` (for Authentication)                                                              |

## Contact

Yash Khambhatta - [yash456k@gmail.com](mailto:yash456k@gmail.com)
