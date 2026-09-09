# Dalis Agenda - Database Design

## 1. Overview

This document defines the initial PostgreSQL database structure for Dalis Agenda.

The database is designed around a multi-tenant architecture where each business operates independently while sharing the same application and database infrastructure.

The initial database focuses only on the entities required for the first version of the product.

---

## 2. General Conventions

### Primary Keys

All primary keys will use UUIDs.

Example:

```text
id UUID PRIMARY KEY
```

UUIDs are preferred because they:

- Avoid predictable sequential identifiers
- Work well in distributed systems
- Are suitable for public APIs
- Reduce coupling between different environments

---

### Naming

Database objects will use `snake_case`.

Examples:

```text
staff_members
staff_member_services
created_at
tenant_id
```

Application entities may use PascalCase:

```text
StaffMember
StaffMemberService
CreatedAt
TenantId
```

---

### Timestamps

Date and time values that represent an exact moment should use PostgreSQL:

```text
TIMESTAMPTZ
```

Examples:

```text
appointments.start_at
appointments.end_at
blocked_times.start_at
```

The backend should store timestamps in UTC.

Each tenant will have its own timezone configuration for displaying and calculating local schedules.

---

### Money

Prices will use:

```text
NUMERIC(12,2)
```

instead of floating-point types.

Example:

```text
12000.00
```

---

## 3. Tenants

Represents each business using Dalis Agenda.

### Table

```text
tenants
```

### Columns

```text
id                  UUID PRIMARY KEY

name                VARCHAR(150) NOT NULL
slug                VARCHAR(100) NOT NULL

business_type       VARCHAR(50) NULL

phone               VARCHAR(30) NULL
email               VARCHAR(255) NULL
address             VARCHAR(255) NULL

timezone            VARCHAR(100) NOT NULL

is_active           BOOLEAN NOT NULL DEFAULT TRUE

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Constraints

```text
UNIQUE (slug)
```

The slug identifies the public booking page.

Example:

```text
dalisagenda.com/barberia-central
```

where:

```text
slug = barberia-central
```

### Notes

`business_type` may contain values such as:

```text
barbershop
beauty_salon
nail_salon
dental_clinic
medical_office
generic
```

It should not affect the core database model.

It may later be used to customize terminology, defaults, or UI behavior.

---

## 4. Users

Represents authenticated users with access to the private application.

### Table

```text
users
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL

email               VARCHAR(255) NOT NULL
password_hash       TEXT NOT NULL

first_name          VARCHAR(100) NOT NULL
last_name           VARCHAR(100) NOT NULL

role                VARCHAR(30) NOT NULL

is_active           BOOLEAN NOT NULL DEFAULT TRUE

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id
```

### Constraints

```text
UNIQUE (tenant_id, email)
```

Possible initial roles:

```text
Owner
Admin
Staff
```

### Notes

A user represents authentication and application access.

A user does not necessarily represent a professional who receives appointments.

---

## 5. Staff Members

Represents professionals who can perform services and receive appointments.

### Table

```text
staff_members
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL
user_id             UUID NULL

first_name          VARCHAR(100) NOT NULL
last_name           VARCHAR(100) NOT NULL

phone               VARCHAR(30) NULL
email               VARCHAR(255) NULL

description         TEXT NULL

is_active           BOOLEAN NOT NULL DEFAULT TRUE

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id

user_id → users.id
```

### Constraints

If a staff member is connected to a user account:

```text
UNIQUE (user_id)
```

### Notes

`user_id` is nullable because a professional may exist without having application access.

Examples:

```text
Business owner
→ has User
→ may not have StaffMember

Employee
→ has StaffMember
→ may not have User

Employee with application access
→ has StaffMember
→ has User
```

---

## 6. Services

Represents services offered by a tenant.

### Table

```text
services
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL

name                VARCHAR(150) NOT NULL
description         TEXT NULL

duration_minutes    INTEGER NOT NULL

price               NUMERIC(12,2) NOT NULL

is_active           BOOLEAN NOT NULL DEFAULT TRUE

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id
```

### Constraints

```text
CHECK (duration_minutes > 0)

CHECK (price >= 0)
```

### Example

```text
name: Haircut
duration_minutes: 30
price: 12000.00
```

---

## 7. Staff Member Services

Represents which services each staff member can perform.

This resolves the many-to-many relationship between Staff Members and Services.

### Table

```text
staff_member_services
```

### Columns

```text
staff_member_id     UUID NOT NULL
service_id          UUID NOT NULL

created_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
staff_member_id → staff_members.id

service_id → services.id
```

### Primary Key

Composite primary key:

```text
PRIMARY KEY (staff_member_id, service_id)
```

This prevents assigning the same service to the same staff member multiple times.

### Example

```text
Juan → Haircut
Juan → Beard Trim
Lucas → Haircut
```

---

## 8. Customers

Represents customers who book appointments.

### Table

```text
customers
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL

first_name          VARCHAR(100) NOT NULL
last_name           VARCHAR(100) NULL

phone               VARCHAR(30) NULL
email               VARCHAR(255) NULL

notes               TEXT NULL

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id
```

### Notes

Customers will not require authentication in V1.

At least one contact method should eventually be required by the application:

```text
phone
OR
email
```

This validation may initially be handled by the backend rather than the database.

Customer deduplication should also be handled carefully.

For example, the system may later identify existing customers through:

```text
tenant_id + phone
```

or:

```text
tenant_id + email
```

but this should not initially be enforced too aggressively at database level.

---

## 9. Availability Rules

Represents the regular weekly working schedule of a staff member.

### Table

```text
availability_rules
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL
staff_member_id     UUID NOT NULL

day_of_week         SMALLINT NOT NULL

start_time          TIME NOT NULL
end_time            TIME NOT NULL

is_active           BOOLEAN NOT NULL DEFAULT TRUE

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id

staff_member_id → staff_members.id
```

### Constraints

```text
CHECK (day_of_week BETWEEN 0 AND 6)

CHECK (start_time < end_time)
```

Suggested day representation:

```text
0 = Sunday
1 = Monday
2 = Tuesday
3 = Wednesday
4 = Thursday
5 = Friday
6 = Saturday
```

### Example

Juan works:

```text
Monday
09:00 - 13:00
14:00 - 18:00
```

That would create two records:

```text
day_of_week: 1
start_time: 09:00
end_time: 13:00
```

and:

```text
day_of_week: 1
start_time: 14:00
end_time: 18:00
```

---

## 10. Blocked Times

Represents exceptional periods where a staff member cannot receive appointments.

### Table

```text
blocked_times
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL
staff_member_id     UUID NOT NULL

start_at            TIMESTAMPTZ NOT NULL
end_at              TIMESTAMPTZ NOT NULL

reason              VARCHAR(255) NULL

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id

staff_member_id → staff_members.id
```

### Constraints

```text
CHECK (start_at < end_at)
```

### Examples

```text
Lunch break

Vacation

Personal appointment

Temporary absence
```

---

## 11. Appointments

Represents appointments booked by customers.

This is one of the most important tables in Dalis Agenda.

### Table

```text
appointments
```

### Columns

```text
id                  UUID PRIMARY KEY

tenant_id           UUID NOT NULL

staff_member_id     UUID NOT NULL
service_id          UUID NOT NULL
customer_id         UUID NOT NULL

start_at            TIMESTAMPTZ NOT NULL
end_at              TIMESTAMPTZ NOT NULL

status              VARCHAR(30) NOT NULL

notes               TEXT NULL

created_at          TIMESTAMPTZ NOT NULL
updated_at          TIMESTAMPTZ NOT NULL
```

### Foreign Keys

```text
tenant_id → tenants.id

staff_member_id → staff_members.id

service_id → services.id

customer_id → customers.id
```

### Constraints

```text
CHECK (start_at < end_at)
```

Possible statuses:

```text
Pending
Confirmed
Completed
Cancelled
NoShow
```

### Important Rule

Two active appointments for the same staff member must never overlap.

This rule should primarily be enforced by the application's booking logic.

Concurrency protection should also be considered so that two customers cannot book the same slot simultaneously.

---

## 12. Entity Relationship Overview

```text
tenants
│
├── users
│
├── staff_members
│   │
│   ├── availability_rules
│   ├── blocked_times
│   ├── staff_member_services
│   └── appointments
│
├── services
│   │
│   ├── staff_member_services
│   └── appointments
│
├── customers
│   │
│   └── appointments
│
└── appointments
```

---

## 13. Relationship Cardinality

### Tenant → Users

```text
1 : N
```

One tenant can have many users.

---

### Tenant → Staff Members

```text
1 : N
```

---

### Tenant → Services

```text
1 : N
```

---

### Tenant → Customers

```text
1 : N
```

---

### Staff Member ↔ Services

```text
N : N
```

Implemented with:

```text
staff_member_services
```

---

### Staff Member → Appointments

```text
1 : N
```

---

### Customer → Appointments

```text
1 : N
```

---

### Service → Appointments

```text
1 : N
```

---

### Staff Member → Availability Rules

```text
1 : N
```

---

### Staff Member → Blocked Times

```text
1 : N
```

---

## 14. Indexes

Indexes should be added according to the most common queries.

### Users

```text
INDEX users_tenant_id_idx
ON users (tenant_id)
```

Authentication will frequently search by:

```text
tenant_id + email
```

The unique constraint already creates an appropriate index.

---

### Staff Members

```text
INDEX staff_members_tenant_id_idx
ON staff_members (tenant_id)
```

---

### Services

```text
INDEX services_tenant_id_idx
ON services (tenant_id)
```

---

### Customers

```text
INDEX customers_tenant_id_idx
ON customers (tenant_id)
```

Possible future indexes:

```text
(tenant_id, phone)

(tenant_id, email)
```

---

### Availability Rules

```text
INDEX availability_rules_staff_day_idx
ON availability_rules (staff_member_id, day_of_week)
```

This helps retrieve the professional's working schedule for a specific day.

---

### Blocked Times

```text
INDEX blocked_times_staff_start_idx
ON blocked_times (staff_member_id, start_at)
```

---

### Appointments

One of the most important indexes:

```text
INDEX appointments_staff_start_idx
ON appointments (staff_member_id, start_at)
```

Also:

```text
INDEX appointments_tenant_start_idx
ON appointments (tenant_id, start_at)
```

and:

```text
INDEX appointments_customer_id_idx
ON appointments (customer_id)
```

These will support queries such as:

```text
Get today's appointments

Get appointments for a staff member

Get upcoming appointments

Get a customer's appointment history
```

---

## 15. Tenant Isolation

Every tenant-owned table must contain:

```text
tenant_id
```

Tenant-scoped tables include:

```text
users
staff_members
services
customers
appointments
availability_rules
blocked_times
```

Every query performed through the authenticated API must be filtered using the current tenant.

Example:

Incorrect:

```text
Get appointment where id = X
```

Correct:

```text
Get appointment
where id = X
and tenant_id = CurrentTenant
```

An entity ID alone must never be trusted for tenant-owned resources.

---

## 16. Deletion Strategy

Business data should generally not be physically deleted when historical information is important.

For entities such as:

```text
StaffMember
Service
User
```

the initial approach should favor:

```text
is_active = false
```

instead of deleting records.

This prevents breaking historical appointments.

For example, if Juan performed an appointment six months ago, deleting Juan should not remove that historical information.

---

## 17. Appointment Historical Data

Appointments currently reference:

```text
service_id
staff_member_id
customer_id
```

However, service information may change over time.

Example:

```text
Haircut today:
$12,000

Haircut six months later:
$20,000
```

An old appointment should still preserve what the customer was charged at the time.

For this reason, the Appointment entity should eventually store snapshot values.

Recommended fields:

```text
service_name        VARCHAR(150) NOT NULL

service_duration_minutes
                    INTEGER NOT NULL

service_price       NUMERIC(12,2) NOT NULL
```

These values are copied from the Service when the appointment is created.

This allows historical appointments to remain accurate even if the Service is edited later.

For the initial implementation, these snapshot fields should be included in the appointment model.

---

## 18. Updated Appointments Table

Taking historical data into account, the recommended initial version becomes:

```text
appointments
```

```text
id                          UUID PRIMARY KEY

tenant_id                   UUID NOT NULL

staff_member_id             UUID NOT NULL
service_id                  UUID NOT NULL
customer_id                 UUID NOT NULL

service_name                VARCHAR(150) NOT NULL
service_duration_minutes    INTEGER NOT NULL
service_price               NUMERIC(12,2) NOT NULL

start_at                    TIMESTAMPTZ NOT NULL
end_at                      TIMESTAMPTZ NOT NULL

status                      VARCHAR(30) NOT NULL

notes                       TEXT NULL

created_at                  TIMESTAMPTZ NOT NULL
updated_at                  TIMESTAMPTZ NOT NULL
```

With:

```text
CHECK (service_duration_minutes > 0)

CHECK (service_price >= 0)

CHECK (start_at < end_at)
```

---

## 19. Initial Tables

The first version of the database will contain:

```text
tenants

users

staff_members

services

staff_member_services

customers

availability_rules

blocked_times

appointments
```

Total:

```text
9 tables
```

---

## 20. Future Tables

The following tables are intentionally excluded from V1 but may be added later.

### locations

For businesses with multiple branches.

### resources

For rooms, equipment, chairs, courts, or other limited resources.

### notifications

For email and WhatsApp notifications.

### subscriptions

For SaaS billing plans.

### payments

For online payments.

### appointment_services

For appointments containing multiple services.

### refresh_tokens

If authentication later uses refresh tokens.

### audit_logs

For tracking important administrative changes.

---

## 21. Initial Database Scope

The first database implementation should solve these problems correctly:

- Tenant isolation
- Authentication
- Staff management
- Service management
- Staff-service assignments
- Customer management
- Weekly staff availability
- Temporary schedule blocks
- Appointment creation
- Appointment history
- Appointment availability calculation
- Prevention of overlapping appointments

Features outside this scope should not influence the initial schema unless they are required for future compatibility.
