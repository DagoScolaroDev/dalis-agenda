# Dalis Agenda - Domain Model

## 1. Overview

This document defines the core domain entities of Dalis Agenda and the relationships between them.

The goal is to keep the domain generic enough to support different appointment-based businesses such as barbershops, beauty salons, dental clinics, medical offices, nail studios, and other service-based businesses.

---

## 2. Core Entities

### Tenant

Represents a business using Dalis Agenda.

Examples:

- Barbería Central
- Sonrisa Dental
- Studio Nails

A tenant owns and isolates all business-related data.

#### Main Properties

- `id`
- `name`
- `slug`
- `businessType`
- `phone`
- `email`
- `address`
- `timezone`
- `isActive`
- `createdAt`
- `updatedAt`

#### Relationships

A tenant has many:

- Users
- Staff Members
- Services
- Customers
- Appointments
- Availability Rules
- Blocked Times

---

### User

Represents a person who can authenticate and access the private application.

A user is not necessarily the same as a staff member.

For example, a business owner may have a user account without receiving appointments.

A staff member may also exist without having login access.

#### Main Properties

- `id`
- `tenantId`
- `email`
- `passwordHash`
- `firstName`
- `lastName`
- `role`
- `isActive`
- `createdAt`
- `updatedAt`

#### Roles

Initial roles:

- `Owner`
- `Admin`
- `Staff`

#### Relationships

A user:

- Belongs to one Tenant
- May optionally be associated with one Staff Member

---

### StaffMember

Represents a person who can perform services and receive appointments.

Examples:

- Barber
- Dentist
- Doctor
- Nail technician
- Stylist
- Personal trainer

#### Main Properties

- `id`
- `tenantId`
- `userId` nullable
- `firstName`
- `lastName`
- `phone`
- `email`
- `description`
- `isActive`
- `createdAt`
- `updatedAt`

#### Relationships

A staff member:

- Belongs to one Tenant
- May optionally be associated with one User
- Can perform many Services
- Has many Availability Rules
- Has many Blocked Times
- Has many Appointments

---

### Service

Represents a service offered by a business.

Examples:

- Haircut
- Beard trim
- Dental consultation
- Nail treatment
- Massage session

#### Main Properties

- `id`
- `tenantId`
- `name`
- `description`
- `durationMinutes`
- `price`
- `isActive`
- `createdAt`
- `updatedAt`

#### Relationships

A service:

- Belongs to one Tenant
- Can be performed by many Staff Members
- Can have many Appointments

---

### StaffMemberService

Represents the relationship between Staff Members and Services.

A staff member may perform multiple services, and one service may be performed by multiple staff members.

This is a many-to-many relationship.

#### Main Properties

- `staffMemberId`
- `serviceId`

#### Example

Juan can perform:

- Haircut
- Beard trim

Lucas can perform:

- Haircut

The Haircut service can therefore be performed by both Juan and Lucas.

---

### Customer

Represents a customer who books appointments.

Customers do not require an authenticated account in the first version.

#### Main Properties

- `id`
- `tenantId`
- `firstName`
- `lastName`
- `phone`
- `email`
- `notes`
- `createdAt`
- `updatedAt`

#### Relationships

A customer:

- Belongs to one Tenant
- Has many Appointments

---

### Appointment

Represents a scheduled appointment.

This is one of the main entities of the system.

#### Main Properties

- `id`
- `tenantId`
- `staffMemberId`
- `serviceId`
- `customerId`
- `startAt`
- `endAt`
- `status`
- `notes`
- `createdAt`
- `updatedAt`

#### Appointment Status

Possible statuses:

- `Pending`
- `Confirmed`
- `Completed`
- `Cancelled`
- `NoShow`

#### Relationships

An appointment:

- Belongs to one Tenant
- Belongs to one Staff Member
- Belongs to one Service
- Belongs to one Customer

---

## 3. Availability

### AvailabilityRule

Represents the regular weekly availability of a Staff Member.

It defines when a staff member normally works.

#### Main Properties

- `id`
- `tenantId`
- `staffMemberId`
- `dayOfWeek`
- `startTime`
- `endTime`
- `isActive`

#### Example

A staff member may have:

Monday:

- 09:00 - 13:00
- 14:00 - 18:00

Tuesday:

- 10:00 - 18:00

Wednesday:

- No availability

Multiple availability rules may exist for the same day.

#### Relationships

An availability rule:

- Belongs to one Tenant
- Belongs to one Staff Member

---

### BlockedTime

Represents a specific period where a Staff Member is unavailable.

Blocked times override regular availability.

#### Main Properties

- `id`
- `tenantId`
- `staffMemberId`
- `startAt`
- `endAt`
- `reason`
- `createdAt`

#### Examples

- Lunch break
- Personal appointment
- Vacation
- Holiday
- Medical leave
- Temporary unavailability

#### Relationships

A blocked time:

- Belongs to one Tenant
- Belongs to one Staff Member

---

## 4. Main Relationships

The core relationships can be represented as:

```text
Tenant
│
├── Users
│
├── StaffMembers
│   │
│   ├── StaffMemberServices
│   ├── AvailabilityRules
│   ├── BlockedTimes
│   └── Appointments
│
├── Services
│   ├── StaffMemberServices
│   └── Appointments
│
├── Customers
│   └── Appointments
│
└── Appointments
```

---

## 5. Relationship Summary

### Tenant → User

One-to-many.

One tenant can have multiple users.

---

### Tenant → StaffMember

One-to-many.

One tenant can have multiple staff members.

---

### Tenant → Service

One-to-many.

One tenant can offer multiple services.

---

### Tenant → Customer

One-to-many.

One tenant can have multiple customers.

---

### StaffMember ↔ Service

Many-to-many.

Implemented through:

`StaffMemberService`

---

### StaffMember → Appointment

One-to-many.

A staff member can receive multiple appointments.

---

### Customer → Appointment

One-to-many.

A customer can have multiple appointments.

---

### Service → Appointment

One-to-many.

A service can appear in multiple appointments.

---

### StaffMember → AvailabilityRule

One-to-many.

A staff member can have multiple weekly availability periods.

---

### StaffMember → BlockedTime

One-to-many.

A staff member can have multiple blocked periods.

---

## 6. Appointment Availability Rules

An appointment can only be created when all the following conditions are true:

1. The tenant is active.
2. The selected service is active.
3. The selected staff member is active.
4. The staff member can perform the selected service.
5. The requested time is inside the staff member's availability.
6. The full service duration fits inside the available period.
7. The requested time does not overlap an existing active appointment.
8. The requested time does not overlap a blocked period.

Example:

Staff availability:

```text
09:00 - 13:00
```

Service duration:

```text
45 minutes
```

Existing appointment:

```text
10:00 - 10:30
```

A new appointment starting at `09:30` would not be available because it would finish at `10:15` and overlap the existing appointment.

---

## 7. Tenant Isolation

Every tenant-owned entity must include a `tenantId`.

Examples:

- StaffMember
- Service
- Customer
- Appointment
- AvailabilityRule
- BlockedTime

Every authenticated operation must be scoped to the current tenant.

The system must never allow a user belonging to one tenant to access or modify data belonging to another tenant.

---

## 8. User and Staff Member Separation

`User` and `StaffMember` must remain separate entities.

This allows scenarios such as:

### Business Owner

Has:

- User account

Does not necessarily have:

- Staff Member profile

### Employee Without Login Access

Has:

- Staff Member profile

Does not have:

- User account

### Employee With Login Access

Has:

- User account
- Staff Member profile

The Staff Member can optionally reference the User through `userId`.

---

## 9. Future Domain Extensions

The model should allow future entities such as:

### Location

For businesses with multiple branches.

Example:

- Palermo branch
- Belgrano branch

---

### Resource

For appointments requiring something in addition to a staff member.

Examples:

- Dental room
- Massage room
- Laser machine
- Sports court

---

### Subscription

For managing Dalis Agenda SaaS plans.

---

### Notification

For:

- Email reminders
- WhatsApp reminders
- Appointment confirmations

---

### Payment

For online appointment payments.

---

### AppointmentService

Could replace the direct `serviceId` on Appointment in the future if one appointment needs to contain multiple services.

Example:

- Haircut
- Beard trim
- Hair treatment

within the same appointment.

---

## 10. Initial Domain Scope

The initial implementation will focus on:

- Tenant
- User
- StaffMember
- Service
- StaffMemberService
- Customer
- Appointment
- AvailabilityRule
- BlockedTime

Other entities should only be introduced when required by future features.
