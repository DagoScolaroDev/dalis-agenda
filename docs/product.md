# Dalis Agenda - Product Definition

## 1. Product Overview

Dalis Agenda is a multi-tenant appointment scheduling platform designed for service-based businesses.

The platform allows businesses to manage their services, staff members, working schedules, customers, and appointments from a single system.

Customers can access a public booking page, choose a service, select an available professional, pick a date and time, and book an appointment online.

The platform is designed to support different business types without being tied to a specific industry.

---

## 2. Target Businesses

Dalis Agenda should be suitable for businesses such as:

- Barbershops
- Hair salons
- Beauty salons
- Nail salons
- Dental clinics
- Medical offices
- Massage and wellness centers
- Personal trainers
- Tattoo studios
- Other appointment-based businesses

The first version should remain generic enough to support multiple industries without requiring different core applications.

---

## 3. User Roles

### Business Owner / Admin

The business owner manages the configuration and daily operation of the business.

Main capabilities:

- Manage business information
- Manage staff members
- Manage services
- Configure working schedules
- View all appointments
- Create, reschedule, and cancel appointments
- Manage customers
- View basic business statistics

### Staff Member

A staff member is a professional who can receive appointments.

Examples include barbers, doctors, dentists, nail technicians, and stylists.

Main capabilities:

- View personal appointments
- View personal schedule
- Block unavailable time slots
- Update appointment status

### Customer

A customer books services through the public booking page.

Main capabilities:

- View available services
- Select a professional
- View available dates and times
- Book an appointment
- Cancel or reschedule an appointment when allowed

---

## 4. Multi-Tenant Model

Dalis Agenda will use a multi-tenant architecture.

Each business will be represented as a tenant.

A tenant has its own:

- Business information
- Staff members
- Services
- Customers
- Appointments
- Schedules
- Configuration

Data from one tenant must never be accessible by another tenant.

Example tenants:

- Barbería Central
- Sonrisa Dental
- Studio Nails
- Centro Estético Norte

All of them use the same Dalis Agenda platform while keeping their data isolated.

---

## 5. MVP Features

The first version of Dalis Agenda will include:

### Authentication

- Business owner registration
- Login
- Logout
- JWT-based authentication

### Business Management

- Create business
- Edit business information
- Configure business name, contact information and basic settings

### Staff Management

- Create staff members
- Edit staff members
- Enable or disable staff members
- Assign services to staff members

### Service Management

- Create services
- Edit services
- Configure service name
- Configure duration
- Configure price
- Enable or disable services

### Schedule Management

- Configure working days
- Configure working hours
- Support different schedules for each staff member
- Block specific periods of time

### Appointment Management

- Create appointments
- View appointments
- Reschedule appointments
- Cancel appointments
- Update appointment status

### Public Booking

Customers will be able to:

1. Access the public page of a business
2. Select a service
3. Select a staff member or choose any available staff member
4. Select a date
5. View available time slots
6. Select a time
7. Enter personal information
8. Confirm the appointment

### Customer Management

- Store customer information
- View customer appointment history
- Search customers

### Dashboard

The initial dashboard may include:

- Appointments today
- Upcoming appointments
- Completed appointments
- Cancelled appointments
- Total customers

---

## 6. Booking Flow

The default booking flow will be:

Service

↓

Staff Member

↓

Date

↓

Available Time Slot

↓

Customer Information

↓

Appointment Confirmation

The system must calculate available time slots based on:

- Staff working schedule
- Service duration
- Existing appointments
- Blocked periods
- Business configuration

---

## 7. Appointment Statuses

Appointments may have the following statuses:

- Pending
- Confirmed
- Completed
- Cancelled
- No-show

The exact status flow can be refined during implementation.

---

## 8. Business Rules

### Appointment Availability

An appointment can only be created if the selected staff member is available for the complete duration of the service.

Appointments must not overlap.

### Service Duration

Each service must define its duration in minutes.

Example:

- Haircut: 30 minutes
- Haircut and beard: 45 minutes
- Dental consultation: 30 minutes

### Staff Availability

Each staff member can have their own weekly schedule.

Example:

Monday:
09:00 - 13:00
14:00 - 18:00

Tuesday:
10:00 - 18:00

Wednesday:
Unavailable

### Blocked Time

Staff members or administrators can block periods where appointments cannot be booked.

Possible reasons include:

- Break
- Personal appointment
- Vacation
- Holiday
- Other

### Tenant Isolation

Every tenant-specific operation must validate the current tenant.

Users from one business must never access resources belonging to another business.

---

## 9. Future Features

The architecture should allow future implementation of:

- Multiple business locations
- Email notifications
- WhatsApp notifications
- Appointment reminders
- Online payments
- Subscription plans
- Advanced analytics
- Custom branding
- Multiple booking page designs
- Industry-specific configurations
- Resource booking
- Waiting lists
- Recurring appointments
- Coupons and promotions
- Calendar integrations

These features are not required for the first version.

---

## 10. Out of Scope for V1

The first version will not include:

- Online payments
- Subscription billing
- WhatsApp integration
- Medical records
- Inventory management
- Accounting
- Advanced marketing tools
- Complex industry-specific workflows

The objective of V1 is to build a reliable and reusable appointment scheduling core.
