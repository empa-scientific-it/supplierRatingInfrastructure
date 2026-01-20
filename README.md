# SupplierRating-Infrastructure

Welcome to the infrastructure repository for the Supplier Rating Software. This project serves as the orchestration layer that ties together the [SupplierRatingFrontend](https://github.com/SupplierRatingSoftware/supplierRatingFrontend) and [SupplierRatingBackend](https://github.com/SupplierRatingSoftware/supplierRatingBackend) applications. It utilizes Docker to manage the containerized deployment of the full stack.

## 1. Key Features
---
1.  **Centralized Orchestration**: Manages the lifecycle of both frontend and backend services simultaneously using Docker Compose.
2.  **Network Configuration**: Establishes a dedicated bridge network (`rating-network`) to ensure secure and direct communication between the frontend and backend containers.
3.  **Production Readiness**: Includes a production-ready example configuration (`docker-compose-prod-example.yml`) that utilizes pre-built images from the container registry.
4.  **Environment Management**: Supports configuration via environment files to handle sensitive credentials and settings.

## 2. Technologies Used
---
1.  **Docker**: Used for containerizing the application services to ensure consistency across different environments.
2.  **Docker Compose**: Used to define and run the multi-container Docker application.
3.  **Nginx**: Serves the frontend application (implied by the frontend service configuration).
4.  **Java/Spring Boot**: Powers the backend API (implied by the backend service context).

## 3. Getting Started
---
Follow these instructions to set up the infrastructure on your local machine.

### 3.1 How to configure openBIS
The backend service requires a connection to an openBIS instance. You must ensure that the connection details are provided in your environment configuration files. Please refer to the Backend documentation for specific API user requirements and property types.

### 3.2 Prerequisites
1.  Ensure you have **Docker** installed on your machine.
2.  Ensure you have **Docker Compose** installed.

### 3.3 Environment Variables
The application relies on environment variables for configuration.
1. The `docker-compose-prod-example.yml` file references a `stack.env` file.
2. Please store this file as a: `docker-compose.yml` in your desired directory.
3.  You should check these files to configure database credentials, openBIS URLs, and other runtime settings. An example `.env` file is in the backend project.

### 3.4 Configuration
1.  Review the `.env` or `stack.env` files.
2.  Update the values to match your local or production environment settings.

### 3.5 Installation
1.  Open your terminal in the root directory of the `docker-compose.yml`.
2.  Run the following command to build and start the services:
    ```bash
    docker-compose up -d
    ```

### 3.6 Usage
1.  Once the containers are running, the services will be accessible at the following ports:
    * **Frontend**: `http://localhost:8082`
    * **Backend API**: `http://localhost:8085` (if you need it outside the container)

## 3. Contributing
We welcome contributions to the infrastructure setup. Please ensure any changes to the Docker configuration are tested against production compose files.

## 3. License
This project is licensed under the MIT License.
Copyright (c) 2025 SupplierRatingSoftware.

Permission is granted to use, copy, modify, and distribute this software under the terms of the MIT License.
