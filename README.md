# FClub Fitness Management System

![FClub Logo](https://ucarecdn.com/b3501875-1c4e-4e68-8641-493ccbdba71f/ChatGPTImage7202514_37_51fotorbgremover20250508235426.png)

FClub is a comprehensive microservice-based solution designed for modern fitness club management. The system provides end-to-end automation of club operations including member management, access control, and notifications through a distributed architecture that ensures scalability and reliability.

## System Architecture

The solution follows a microservices pattern with 4 core backend services and 2 client applications, orchestrated through Kubernetes. All external requests are routed via Nginx ingress controller. Internal communication utilizes both HTTP REST APIs for service-to-service interactions and RabbitMQ for asynchronous messaging to the logging service. The logging implementation incorporates Redis for caching user's logs.

![FClub Architecture Diagram](https://ucarecdn.com/8326f90c-59f5-490f-9718-44e18d88eec5/20250513173612.png)

Each microservice adheres to **Clean Architecture** principles, enforcing a strict separation of concerns through modular layers:
- **Domain Layer**: Contains core business logic, entities, and interfaces.  
- **Application Layer**: Implements use cases.  
- **Infrastructure Layer**: Handles external concerns (database access, messaging, APIs) via adapters.  
- **WebUI Layer (Presentation)**: Delivers HTTP endpoints (controllers, middleware) and serves as the entry point for external requests.  
- **Shared Layer**: Hosts cross-cutting utilities and common contracts consumed by other layers.  

Dependencies flow inward: outer layers (WebUI, Infrastructure) depend on inner layers (Application, Domain), while the **Domain layer remains isolated** from all external concerns. This ensures testability, flexibility, and maintainability by decoupling business rules from implementation details. 

![FClub Architecture Diagram Microservice](https://ucarecdn.com/e228e111-e652-4523-ade6-77d3a7ba0ba6/20250513174442.png)

## Core Components

| Component                | Description                                                                                     | Technology Stack                          |
|--------------------------|-------------------------------------------------------------------------------------------------|-------------------------------------------|
| Management Service       | Central business logic service handling main domain entities  | ASP.NET Core, C#, PostgreSQL, RabbitMQ|
| Access Control Service   | Access validation system integrating with turnstile hardware      | ASP.NET Core, C#, PostgreSQL, RabbitMQ              |
| Notification Service     | Manages email notifications for clients   | ASP.NET Core, C#, PostgreSQL, RabbitMQ      |
| Logging Service     | Centralized logging system for all services with message queuing and caching   | ASP.NET Core, C#, PostgreSQL, RabbitMQ, Redis      |
| Admin Dashboard          | Responsive management interface for staff with role-based access control                 | React, TypeScript       |
| Turnstile Interface      | Simulation of the club's access control system          | React, TypeScript           |
| Message Broker      | Handles asynchronous logs communication between services         |  RabbitMQ          |
| Cache      | Provides caching capabilities for logs         | Redis           |

## Security Architecture

The system implements a robust **JWT-based authentication and authorization** mechanism with **role-based access control (RBAC)**. Each microservice validates incoming requests by verifying the `Bearer` JWT token in the `Authorization` header.

### Key Security Features

#### Strict Token Validation
- **All requests** (client-to-server and server-to-server) require a valid JWT.
- Tokens are signed with **different keys** for:
  - External requests (client-facing)
  - Internal service communication (server-to-server)
- Validation includes:
  - Issuer (`iss`) verification
  - Expiration time (`exp`) checks

#### Two-Tier Token System
| Token Type       | Lifetime | Purpose                          |
|------------------|----------|----------------------------------|
| **Access Token** | 60 days  | Short-lived API authorization    |
| **Refresh Token**| 360 days | Secure renewal of access tokens  |

#### Role-Based Access Control (RBAC)
- JWTs include **role claims** for granular permissions.
- Microservices enforce access based on:
  - User roles (e.g., `Admin`, `Manager`)

## Source Code Repositories

| Repository | Purpose |
|------------|---------|
| [FClub.Backend.CombinedMessaging](https://github.com/denekben/FClub.Backend.CombinedMessaging) | Micsrocesvices backend application |
| [FClub.Backend.Common](https://github.com/denekben/FClub.Backend.Common) | Common microservice class library |
| [FClub.Client.Management.CombinedMessaging](https://github.com/denekben/FClub.Client.Management.CombinedMessaging) | Admin dashboard frontend |
| [FClub.Client.Turnstile](https://github.com/denekben/FClub.Client.Turnstile) | Turnstile access control frontend |
| [FClub.K8S.CombinedMessaging](https://github.com/denekben/FClub.K8S.CombinedMessaging) | Kubernetes deployment configurations |

## Deployment Scheme

[!Deployment](https://ucarecdn.com/a469c658-1e7e-4307-87c7-f13983ea160e/20250603125556.png)

## Deployment Commands

1. Create PostgreSQL secrets:
```bash
kubectl create secret generic postgres-secrets --from-literal=POSTGRES_PASSWORD={YOUR_PASSWORD}
```
2. Deploy Broker:
```bash
kubectl apply -f Server/RabbitMq/rabbitmq-depl.yaml
```

3. Create Persistent Volume Claims:
```bash
kubectl apply -f Server/Notifications/notifications-local-pvc.yaml
kubectl apply -f Server/Management/management-local-pvc.yaml
kubectl apply -f Server/AccessControl/accesscontrol-local-pvc.yaml
kubectl apply -f Server/Logging/logging-local-pvc.yaml
kubectl apply -f Server/Logging/logging-redis-local-pvc.yaml
```

4. Deploy PostgreSQL databases:
```bash
kubectl apply -f Server/Notifications/notifications-postgres-depl.yaml
kubectl apply -f Server/Management/management-postgres-depl.yaml
kubectl apply -f Server/AccessControl/accesscontrol-postgres-depl.yaml
kubectl apply -f Server/Logging/logging-postgres-depl.yaml
kubectl apply -f Server/Logging/logging-redis-depl.yaml
```

5. Deploy microservices:
```bash
kubectl apply -f Server/Notifications/notifications-depl.yaml
kubectl apply -f Server/Management/management-depl.yaml
kubectl apply -f Server/AccessControl/accesscontrol-depl.yaml
kubectl apply -f Server/Logging/logging-depl.yaml
```

6. Deploy client applications:
```bash
kubectl apply -f Client/client-management-depl.yaml
kubectl apply -f Client/client-turnstile-depl.yaml
```

7. Set up ingress:
```bash
kubectl apply -f Nginx/ingress-srv.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

## How does it look like
Admin dashboard frontend
![Management-Client](https://ucarecdn.com/e22c606a-9064-4845-9469-331a451a6164/20250509191911.png)
Turnstile access control frontend
![Turnstile-Client](https://ucarecdn.com/d9e270a9-5574-440b-941d-7514da23cb52/20250509191945.png)
