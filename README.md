# Infernum
TFG DAW

This project was realized by [Martin Morente Vargas](https://github.com/martinmorente-dev) & [Cristian Neagu Cocargeanu](https://github.com/CR15T18N)

# Infernum

**Infernum** is a digital game store web platform. It lets users browse a game catalog, search and filter by genre, view game details, purchase games and manage their personal library, all from the browser.

The project is split into three independent repositories: the web application users interact with, the REST API that powers it, and the cloud infrastructure that hosts both.

| Part | Description | Main technologies | Repository |
|------|-------------|-------------------|------------|
| **Frontend** | Single-page web application users interact with | Angular 20, TypeScript, SASS | [`InfernumFrontend`](https://github.com/CR15T18N/InfernumFrontend) |
| **Backend** | REST API and admin panel | PHP 8.4, Laravel, MySQL, Filament | [`InfernumBackend`](https://github.com/martinmorente-dev/InfernumBackend) |
| **Architecture** | Infrastructure as Code that provisions the AWS environment | Terraform, AWS (EC2, NLB, Route 53, CodeDeploy) | [`InfernumArquitecture`](https://github.com/martinmorente-dev/InfernumArquitecture) |


## What does the application do?

### For users (customers)

- **Browse the catalog:** home page with featured games, new releases, top sellers and special offers.
- **Search and filter:** store page with title search, genre filtering and pagination.
- **View game details:** image gallery, description, minimum and recommended system requirements, and the final price with discounts applied.
- **Register and log in:** secure token-based authentication.
- **Shopping cart:** add games to the cart and pay through a payment gateway (Stripe).
- **Personal library:** access to every game the user has purchased.
- **User profile:** view and edit display name, bio and profile picture.

### For administrators

- **Admin panel** (built with Filament) to manage games, genres, discounts, requirements and images.
- Direct access to the panel from the application when logging in with an administrator account.

---

## General architecture

### Application

```
┌──────────────────────┐        HTTP / JSON        ┌──────────────────────┐
│       FRONTEND       │  ───────────────────────► │       BACKEND        │
│   Angular (SPA)      │   /api/v1/...  (Bearer)   │   Laravel (REST API) │
│ Apache + ModSecurity │  ◄─────────────────────── │   Sanctum · Cashier  │
└──────────────────────┘                           └──────────┬───────────┘
                                                              │
                                                   ┌──────────▼───────────┐
                                                   │   MySQL 8  ·  Stripe │
                                                   └──────────────────────┘
```

1. The user interacts with the **frontend** in their browser.
2. The frontend consumes the backend **REST API** (`/api/v1/`), sending the authentication token with every request.
3. The backend handles business logic, the MySQL database and Stripe payments.

### Cloud infrastructure (AWS)

The whole environment is described in code and deployed on AWS with Terraform:

```
Internet ──(80 / 443)──► [Network Load Balancer]
                              │            │
                              ▼            ▼
                          [Front 1]    [Front 2]     Frontend (Docker)
                                 │
                                 │ (port 80, internal only)
                                 ▼
                            [Backend]                 Laravel API (Docker)
                                 ▲
                                 │ (SSH, internal only)
Admin ──(SSH)──► [Bastion Host] ─┘
```

- **Network Load Balancer** distributes HTTP/HTTPS traffic between two frontend servers.
- **Frontend servers (x2)** run the Angular application behind Apache in Docker containers.
- **Backend server** runs the Laravel API and is reachable only from the frontend servers.
- **Bastion host** is the only entry point for administrative SSH access.
- **Route 53** manages the DNS records and **AWS CodeDeploy** automates application deployments.

---

## Technologies

**Frontend**
- Angular 20 (standalone components, Signals) · TypeScript · SASS
- Production server: Apache 2 with ModSecurity (WAF)

**Backend**
- PHP 8.4 · Laravel · MySQL 8
- Laravel Sanctum (token authentication) · Laravel Cashier (Stripe)
- Filament v3 (admin panel) · L5 Swagger (OpenAPI documentation)

**Infrastructure**
- Terraform (remote state in S3)
- AWS: EC2 (Ubuntu 22.04), Network Load Balancer, Route 53, CodeDeploy, SSM Parameter Store
- Docker and Docker Compose
- CI/CD with GitHub Actions and AWS CodeDeploy

---

## Getting started

Each repository contains its own detailed documentation for installation, development and deployment. In short:

1. Clone the repositories:
   ```bash
   git clone https://github.com/<user>/InfernumArquitecture.git
   git clone https://github.com/<user>/InfernumBackend.git
   git clone https://github.com/<user>/InfernumFrontend.git
   ```
2. **Cloud deployment:** provision the AWS infrastructure first with Terraform (`terraform init && terraform apply` inside the `deploy/` folder of the architecture repository). Afterwards, application code is deployed to the servers through AWS CodeDeploy.
3. **Local development:** start the **backend** first (Docker Compose, migrations and seeders) following its README, then the **frontend** (`ng serve` or Docker Compose). In development, `/api` requests are forwarded to the backend through a proxy.

> Both Docker development environments share a network (`shared_network`) so they can communicate with each other.

---

## Detailed documentation

- 📘 **Frontend:** [InfernumFrontend README](https://github.com/CR15T18N/InfernumFrontend)
- 📗 **Backend:** [InfernumBackend README](https://github.com/martinmorente-dev/InfernumBackend)
- 📙 **Architecture:** [InfernumArquitecture README](https://github.com/martinmorente-dev/InfernumArquitecture)
- 📄 **API documentation:** available through Swagger in the backend.

---

## Authors

Project developed by **Martin Morente Vargas & Cristian Neagu Cocargeanu**.
