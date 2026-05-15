# 🚦 Urban Traffic Management Platform

> **Mini Projet – Web Services & GraphQL**  
> Plateforme intelligente de gestion du trafic urbain basée sur une architecture microservices avec API Gateway GraphQL.

---

## 📋 Table des matières

- [Vue d'ensemble](#vue-densemble)
- [Architecture](#architecture)
- [Services](#services)
- [Stack Technique](#stack-technique)
- [Prérequis](#prérequis)
- [Installation & Lancement](#installation--lancement)
- [Variables d'environnement](#variables-denvironnement)
- [API GraphQL](#api-graphql)
- [Authentification](#authentification)
- [Structure du projet](#structure-du-projet)
- [Base de données](#base-de-données)
- [Tests](#tests)
- [Docker](#docker)
- [Diagrammes UML](#diagrammes-uml)
- [Collection Postman](#collection-postman)
- [Contributeurs](#contributeurs)

---

## Vue d'ensemble

Cette plateforme distributed permet la **supervision des véhicules**, la **détection des incidents** et l'**analyse de la circulation** en temps réel dans un contexte urbain.

Elle est composée de **5 microservices indépendants** exposés via une **API Gateway GraphQL** unique.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   CLIENT (React/Next.js)             │
└──────────────────────┬──────────────────────────────┘
                       │ GraphQL / WebSocket
┌──────────────────────▼──────────────────────────────┐
│              API GATEWAY (GraphQL)                   │
│         Apollo Federation / Schema Stitching         │
└──┬────────┬─────────┬──────────┬────────────────────┘
   │        │         │          │          │
   ▼        ▼         ▼          ▼          ▼
┌──────┐ ┌──────┐ ┌──────┐ ┌────────┐ ┌──────────────┐
│ Auth │ │Véhi- │ │Tra-  │ │Incident│ │Notifications │
│Serv. │ │cules │ │fic   │ │Service │ │  Service     │
└──┬───┘ └──┬───┘ └──┬───┘ └────┬───┘ └──────┬───────┘
   │        │        │          │             │
   ▼        ▼        ▼          ▼             ▼
┌─────────────────────────────────────────────────────┐
│              PostgreSQL / MySQL Databases            │
└─────────────────────────────────────────────────────┘
```

---

## Services

### 1. 🔐 Service Authentification — `auth-service` (port `3001`)

Gère les identités et les accès à la plateforme.

| Fonctionnalité | Description |
|---|---|
| Inscription | Création d'un compte utilisateur |
| Connexion | Génération d'un token JWT signé |
| Rôles | `ADMIN`, `OPERATOR` |
| Sécurité | Hachage bcrypt, tokens d'expiration |

**Mutations GraphQL :**
```graphql
mutation Register($input: RegisterInput!) {
  register(input: $input) { token user { id email role } }
}

mutation Login($input: LoginInput!) {
  login(input: $input) { token user { id email role } }
}
```

---

### 2. 🚗 Service Gestion des Véhicules — `vehicle-service` (port `3002`)

Supervision et suivi GPS des véhicules.

| Fonctionnalité | Description |
|---|---|
| Ajouter un véhicule | Enregistrement d'un nouveau véhicule |
| Lister les véhicules | Consultation de tous les véhicules |
| Détail véhicule | Informations complètes d'un véhicule |
| Position GPS | Enregistrement de positions simulées |
| Historique | Consultation des déplacements passés |

**Queries & Mutations GraphQL :**
```graphql
query GetVehicles { vehicles { id licensePlate model status } }

query GetVehicle($id: ID!) { vehicle(id: $id) { id licensePlate gpsHistory { lat lng timestamp } } }

mutation AddVehicle($input: VehicleInput!) { addVehicle(input: $input) { id } }

mutation RecordGPS($vehicleId: ID!, $lat: Float!, $lng: Float!) {
  recordPosition(vehicleId: $vehicleId, lat: $lat, lng: $lng) { id timestamp }
}
```

---

### 3. 🗺️ Service Gestion du Trafic — `traffic-service` (port `3003`)

Analyse et classification des zones de circulation.

| Fonctionnalité | Description |
|---|---|
| Zones de circulation | Création et gestion des zones |
| Densité du trafic | Mesure en temps réel |
| Détection congestion | Identification des zones saturées |
| Classification | `FAIBLE` / `MOYEN` / `ÉLEVÉ` |

**Queries & Mutations GraphQL :**
```graphql
query GetTrafficZones { trafficZones { id name density level } }

mutation CreateZone($input: ZoneInput!) { createZone(input: $input) { id name } }

mutation UpdateDensity($zoneId: ID!, $density: Float!) {
  updateTrafficDensity(zoneId: $zoneId, density: $density) { id level }
}
```

---

### 4. ⚠️ Service Gestion des Incidents — `incident-service` (port `3004`)

Déclaration et suivi des incidents de circulation.

**Types d'incidents :**
- `ACCIDENT`
- `TRAVAUX`
- `ROUTE_FERMEE`
- `EMBOUTEILLAGE`

**Statuts :**
- `SIGNALE` → `EN_COURS` → `RESOLU`

**Queries & Mutations GraphQL :**
```graphql
query GetIncidents { incidents { id type status location createdAt } }

mutation DeclareIncident($input: IncidentInput!) { declareIncident(input: $input) { id type status } }

mutation UpdateIncidentStatus($id: ID!, $status: IncidentStatus!) {
  updateIncidentStatus(id: $id, status: $status) { id status updatedAt }
}
```

---

### 5. 🔔 Service Notifications — `notification-service` (port `3005`)

Gestion des alertes et notifications utilisateur.

| Fonctionnalité | Description |
|---|---|
| Envoyer notification | Création d'une nouvelle notification |
| Consulter | Liste des notifications d'un utilisateur |
| Marquer comme lue | Mise à jour du statut de lecture |

**Queries & Mutations GraphQL :**
```graphql
query GetNotifications($userId: ID!) {
  notifications(userId: $userId) { id message isRead createdAt }
}

mutation SendNotification($input: NotificationInput!) { sendNotification(input: $input) { id } }

mutation MarkAsRead($id: ID!) { markNotificationAsRead(id: $id) { id isRead } }
```

---

## Stack Technique

| Couche | Technologie |
|---|---|
| Runtime | Node.js 20+ |
| Framework | NestJS |
| API | GraphQL (Apollo Server / Federation) |
| Base de données | PostgreSQL |
| ORM | TypeORM / Prisma |
| Authentification | JWT (jsonwebtoken) + bcrypt |
| Containerisation | Docker & Docker Compose |
| Frontend *(bonus)* | React / Next.js |
| Temps réel *(bonus)* | WebSocket (GraphQL Subscriptions) |
| Tests | Jest |
| CI/CD *(bonus)* | GitHub Actions |

---

## Prérequis

- [Node.js](https://nodejs.org/) v20+
- [npm](https://www.npmjs.com/) ou [yarn](https://yarnpkg.com/)
- [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
- [PostgreSQL](https://www.postgresql.org/) (si lancé sans Docker)
- [Git](https://git-scm.com/)

---

## Installation & Lancement

### Avec Docker (recommandé)

```bash
# Cloner le repo
git clone https://github.com/<your-org>/urban-traffic-platform.git
cd urban-traffic-platform

# Copier les variables d'environnement
cp .env.example .env

# Lancer tous les services
docker-compose up --build
```

L'API Gateway sera accessible sur **`http://localhost:4000/graphql`**.

---

### Sans Docker (développement)

```bash
# Installer les dépendances de chaque service
cd auth-service && npm install
cd ../vehicle-service && npm install
cd ../traffic-service && npm install
cd ../incident-service && npm install
cd ../notification-service && npm install
cd ../gateway && npm install

# Lancer chaque service dans un terminal séparé
npm run start:dev
```

---

## Variables d'environnement

Créer un fichier `.env` à la racine (voir `.env.example`) :

```env
# JWT
JWT_SECRET=your_super_secret_key
JWT_EXPIRES_IN=7d

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=password
DB_NAME=traffic_platform

# Services Ports
AUTH_SERVICE_PORT=3001
VEHICLE_SERVICE_PORT=3002
TRAFFIC_SERVICE_PORT=3003
INCIDENT_SERVICE_PORT=3004
NOTIFICATION_SERVICE_PORT=3005
GATEWAY_PORT=4000
```

---

## API GraphQL

### Endpoint

```
http://localhost:4000/graphql
```

### GraphQL Playground / Apollo Sandbox

Accessible via navigateur sur `http://localhost:4000/graphql`

### Exemple de requête complète

```graphql
# Connexion
mutation {
  login(input: { email: "admin@traffic.com", password: "Admin@123" }) {
    token
    user { id email role }
  }
}

# Obtenir les zones de trafic avec incidents
query {
  trafficZones {
    id
    name
    level
    density
  }
  incidents(status: SIGNALE) {
    id
    type
    location
    status
  }
}
```

---

## Authentification

Toutes les routes protégées nécessitent un **JWT Bearer token** dans les headers HTTP :

```http
Authorization: Bearer <your_jwt_token>
```

### Rôles et permissions

| Action | OPERATOR | ADMIN |
|---|---|---|
| Consulter véhicules | ✅ | ✅ |
| Ajouter véhicule | ✅ | ✅ |
| Déclarer incident | ✅ | ✅ |
| Modifier statut incident | ✅ | ✅ |
| Gérer utilisateurs | ❌ | ✅ |
| Créer zones trafic | ❌ | ✅ |
| Envoyer notifications | ❌ | ✅ |

---

## Structure du projet

```
urban-traffic-platform/
│
├── gateway/                    # API Gateway GraphQL
│   ├── src/
│   │   ├── schema/
│   │   └── resolvers/
│   └── Dockerfile
│
├── auth-service/               # Service Authentification
│   ├── src/
│   │   ├── auth/
│   │   ├── users/
│   │   └── main.ts
│   └── Dockerfile
│
├── vehicle-service/            # Service Véhicules
│   ├── src/
│   │   ├── vehicles/
│   │   ├── gps/
│   │   └── main.ts
│   └── Dockerfile
│
├── traffic-service/            # Service Trafic
│   ├── src/
│   │   ├── zones/
│   │   ├── density/
│   │   └── main.ts
│   └── Dockerfile
│
├── incident-service/           # Service Incidents
│   ├── src/
│   │   ├── incidents/
│   │   └── main.ts
│   └── Dockerfile
│
├── notification-service/       # Service Notifications
│   ├── src/
│   │   ├── notifications/
│   │   └── main.ts
│   └── Dockerfile
│
├── frontend/                   # Dashboard React/Next.js (bonus)
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── hooks/
│   └── Dockerfile
│
├── docs/
│   ├── uml/                    # Diagrammes UML
│   │   ├── class-diagram.puml
│   │   ├── sequence-diagram.puml
│   │   └── use-case.puml
│   └── postman/
│       └── collection.json     # Collection Postman
│
├── docker-compose.yml
├── .env.example
├── .github/
│   └── workflows/
│       └── ci.yml              # GitHub Actions CI/CD
└── README.md
```

---

## Base de données

### Schéma simplifié

```sql
-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role ENUM('ADMIN', 'OPERATOR') DEFAULT 'OPERATOR',
  created_at TIMESTAMP DEFAULT NOW()
);

-- Vehicles
CREATE TABLE vehicles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  license_plate VARCHAR(20) UNIQUE NOT NULL,
  model VARCHAR(100),
  status VARCHAR(50),
  owner_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- GPS Positions
CREATE TABLE gps_positions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  vehicle_id UUID REFERENCES vehicles(id),
  latitude DECIMAL(10, 8) NOT NULL,
  longitude DECIMAL(11, 8) NOT NULL,
  recorded_at TIMESTAMP DEFAULT NOW()
);

-- Traffic Zones
CREATE TABLE traffic_zones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  density DECIMAL(5, 2) DEFAULT 0,
  level ENUM('FAIBLE', 'MOYEN', 'ELEVE') DEFAULT 'FAIBLE',
  coordinates JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Incidents
CREATE TABLE incidents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type ENUM('ACCIDENT', 'TRAVAUX', 'ROUTE_FERMEE', 'EMBOUTEILLAGE') NOT NULL,
  status ENUM('SIGNALE', 'EN_COURS', 'RESOLU') DEFAULT 'SIGNALE',
  location VARCHAR(255),
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  description TEXT,
  declared_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Notifications
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  message TEXT NOT NULL,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Tests

```bash
# Tests unitaires
npm run test

# Tests e2e
npm run test:e2e

# Couverture
npm run test:cov
```

---

## Docker

### docker-compose.yml (aperçu)

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: traffic_platform
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  gateway:
    build: ./gateway
    ports:
      - "4000:4000"
    depends_on: [auth-service, vehicle-service, traffic-service, incident-service, notification-service]

  auth-service:
    build: ./auth-service
    ports:
      - "3001:3001"
    depends_on: [postgres]

  vehicle-service:
    build: ./vehicle-service
    ports:
      - "3002:3002"
    depends_on: [postgres]

  traffic-service:
    build: ./traffic-service
    ports:
      - "3003:3003"
    depends_on: [postgres]

  incident-service:
    build: ./incident-service
    ports:
      - "3004:3004"
    depends_on: [postgres]

  notification-service:
    build: ./notification-service
    ports:
      - "3005:3005"
    depends_on: [postgres]
```

---

## Diagrammes UML

Les diagrammes UML sont disponibles dans le dossier `/docs/uml/` :

- **Diagramme de cas d'utilisation** — `use-case.puml`
- **Diagramme de classes** — `class-diagram.puml`
- **Diagramme de séquence** (ex: déclaration d'un incident) — `sequence-diagram.puml`
- **Diagramme de déploiement** — `deployment.puml`

Utiliser [PlantUML](https://plantuml.com/) ou [draw.io](https://app.diagrams.net/) pour visualiser.

---

## Collection Postman

La collection Postman est disponible dans `/docs/postman/collection.json`.

Elle inclut :
- Toutes les requêtes GraphQL de test
- Variables d'environnement pré-configurées
- Scripts de test automatisés

**Import :**
1. Ouvrir Postman
2. `File > Import`
3. Sélectionner `docs/postman/collection.json`

---

## Requêtes GraphQL de test

```graphql
# 1. Register
mutation { register(input: { email: "test@mail.com", password: "Test@1234", role: OPERATOR }) { token } }

# 2. Login
mutation { login(input: { email: "test@mail.com", password: "Test@1234" }) { token } }

# 3. Add Vehicle
mutation { addVehicle(input: { licensePlate: "TUN-1234", model: "Toyota Corolla" }) { id } }

# 4. Record GPS
mutation { recordPosition(vehicleId: "uuid-here", lat: 36.8065, lng: 10.1815) { id timestamp } }

# 5. Get Traffic Zones
query { trafficZones { id name level density } }

# 6. Declare Incident
mutation { declareIncident(input: { type: ACCIDENT, location: "Avenue Habib Bourguiba", description: "Collision entre 2 véhicules" }) { id status } }

# 7. Update Incident Status
mutation { updateIncidentStatus(id: "uuid-here", status: EN_COURS) { id status } }

# 8. Get Notifications
query { notifications(userId: "uuid-here") { id message isRead createdAt } }
```

---

## Contributeurs

| Nom | Rôle |
|---|---|
| *(Membre 1)* | Auth Service + API Gateway |
| *(Membre 2)* | Vehicle Service + GPS |
| *(Membre 3)* | Traffic Service + Incidents |
| *(Membre 4)* | Notifications + Frontend |

---

## Licence

Ce projet est réalisé dans le cadre du module **Web Services** — Usage académique uniquement.

---

> ⚠️ **Note :** Tout plagiat ou copie entre groupes entraînera une sanction académique. Ce projet doit être réalisé avec sérieux, rigueur et esprit d'ingénierie.
