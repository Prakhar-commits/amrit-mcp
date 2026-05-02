# AMRIT Architecture Overview

## Platform Purpose

AMRIT (Accessible Medical Records via Integrated Technologies) is an open-source EHR platform designed for last-mile healthcare delivery in India. It supports field workers, nurses, and doctors operating in Health & Wellness Centres, Mobile Medical Units, and 104 helpline services.

## Repository Structure

All AMRIT repositories live under https://github.com/PSMRI and follow a naming convention: `AMRIT-<Module>-API` for backends and `AMRIT-<Module>-UI` for frontends.

### Core Repositories

- **AMRIT-HWC-API** — Health & Wellness Centre backend. Handles OPD visits, NCD screening, maternal and child health records.
- **AMRIT-HWC-UI** — Angular frontend for HWC field workers and nurses.
- **AMRIT-Helpline104-API** — Backend for the 104 health helpline. Handles inbound calls, triage, and referrals.
- **AMRIT-TM-API** — Telemedicine module. Video/audio consultations between patients and doctors.
- **AMRIT-MMU-API** — Mobile Medical Unit backend. Similar to HWC but for mobile van deployments.
- **AMRIT-Common-API** — Shared services: beneficiary registration, identity, location master data.
- **AMRIT-Inventory-API** — Drug and equipment inventory management across facilities.

## Backend Architecture

All AMRIT backend services are Java Spring Boot applications. They follow a strict layered architecture:

### Layer pattern

```
REST Controller  →  Service Interface  →  Service Implementation  →  JPA Repository  →  MySQL
```

### Conventions

- Controllers are annotated with `@RestController` and `@RequestMapping`
- Every service has an interface and an implementation class
- Repositories extend `JpaRepository<Entity, Long>` — no raw SQL
- DTOs are used for request/response shapes; entities are not exposed directly
- All APIs return a standard response wrapper: `{ statusCode, data, errorMessage }`

## Frontend Architecture

AMRIT frontends are Angular applications. As of 2024-2025, newer modules use Angular standalone components.

### Conventions

- Component files: `feature-name.component.ts` + `.html` + `.scss`
- HTTP calls go through service classes, never directly in components
- HTTP interceptors handle auth token injection
- Reactive forms are preferred over template-driven forms

## Beneficiary Registration

The beneficiary is the core entity in AMRIT — every patient interaction is linked to a beneficiary record.

### Registration flow

1. Field worker opens AMRIT-HWC-UI and navigates to beneficiary registration
2. A request is made to `AMRIT-Common-API /beneficiary/register`
3. Common API validates and persists the beneficiary record with a unique `beneficiaryID`
4. The `beneficiaryID` is shared across all AMRIT modules — HWC, MMU, 104 etc.

### Key fields

- `beneficiaryID` — unique identifier across the platform
- `beneficiaryRegID` — registration-specific ID per facility visit
- `vanID` — identifies which health delivery point registered them

## 104 Helpline Call Routing

The 104 helpline receives inbound calls from citizens seeking health advice.

### Call flow

1. Call arrives at the telephony system and is assigned to an available health worker
2. The worker opens a new call session in the 104 UI
3. Chief complaint is recorded and a triage algorithm suggests a call category
4. Based on category: advice only, refer to facility, or escalate to a doctor
5. Call summary is saved against the beneficiary record in Common API

## Spring Boot Service Conventions

When writing a new feature in any AMRIT backend:

1. Create the JPA entity in the `entities` package
2. Create the repository interface extending `JpaRepository`
3. Define the service interface in the `service` package
4. Implement it in `service/impl` — business logic lives here
5. Create the controller in the `controller` package
6. Use `@Transactional` on service methods that write to the database
7. Validate inputs using Bean Validation (`@NotNull`, `@Size`, etc.) on DTOs

## SDLC Overview

AMRIT follows a structured SDLC documented at:
https://pmp.piramalswasthya.org/confluence/spaces/AMRIT/pages/76546619/AMRIT+Software+Development+Lifecycle

### Phases

1. **Requirements** — Business requirements captured in Confluence as concept notes or BRDs
2. **Analysis** — Tech team breaks down requirements into JIRA stories and sub-tasks
3. **Development** — Feature branches, PR review against coding standards
4. **QA** — Test cases written against JIRA stories, regression on staging
5. **Deployment** — CI/CD pipeline deploys to staging then production
6. **Support** — L2 team triages bugs, links to JIRA, provides hotfixes

## Open Source Contribution via C4GT

AMRIT participates in Code for GovTech (C4GT). Contributors pick issues from the AMRIT GitHub repo, propose solutions, and work with mentors over ~8-10 weeks. The main challenge for new contributors is understanding how 10+ repositories connect — which API to modify for a given feature, which UI repo calls it, and what the coding conventions are.
