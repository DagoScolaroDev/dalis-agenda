# Dalis Agenda

Dalis Agenda is a multi-tenant appointment scheduling platform designed for service-based businesses.

The platform allows businesses such as barbershops, beauty salons, nail studios, dental clinics, medical offices, and other appointment-based businesses to manage their services, staff, availability, customers, and appointments from a single system.

## Main Goals

* Provide a reusable appointment scheduling system for different business types.
* Support multiple businesses through a multi-tenant architecture.
* Allow customers to book appointments online.
* Allow businesses to manage their staff, services, schedules, and appointments.
* Build a scalable architecture that can support different industries and booking flows in the future.

## MVP

The first version will include:

* Authentication
* Multi-tenant support
* Business configuration
* Staff management
* Service management
* Working schedules
* Appointment availability
* Online appointment booking
* Appointment management
* Customer management
* Basic dashboard

## Tech Stack

### Frontend

* Angular
* TypeScript
* SCSS
* Angular Signals
* Reactive Forms

### Backend

* ASP.NET Core
* C#
* Entity Framework Core
* JWT Authentication

### Database

* PostgreSQL

### Infrastructure

* Docker
* Docker Compose

## Repository Structure

```text
dalis-agenda/
├── apps/
│   ├── web/
│   └── api/
├── docs/
├── docker-compose.yml
└── README.md
```

## Status

Currently in the planning and architecture phase.
