# Database Schema

Schema defined in `.server/db/schema.ts` using Drizzle ORM.

## Core Tables

### users
System users with authentication.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| email | varchar | unique |
| name | varchar | |
| avatarUrl | varchar | |
| isSuperadmin | boolean | grants full system access |
| createdAt, updatedAt | timestamptz | |

### owners
Organizations/promoters managing events.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| name, email, description | varchar | |
| createdAt, updatedAt | timestamptz | |

### memberships
Unified hierarchical permission system.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| userId | uuid | FK → users, cascade delete |
| scopeType | enum | owner/event/entity |
| ownerId | uuid | FK → owners, cascade delete |
| eventId | uuid | FK → events, cascade delete, nullable |
| entityId | uuid | FK → entities, cascade delete, nullable |
| role | enum | read/write |

**Unique constraint:** `(userId, scopeType, ownerId, eventId, entityId)`

### events
Top-level event containers.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| ownerId | uuid | FK → owners, cascade delete |
| venueId | uuid | FK → venues, set null |
| name, description | varchar | |
| startDate, endDate | timestamptz | |
| status | enum | draft/published/archived |
| createdBy | uuid | FK → users |
| createdAt, updatedAt | timestamptz | |

### venues
Physical locations.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| name, address, city, country, postalCode | varchar | |
| latitude, longitude | numeric | for mapping |
| capacity | integer | |
| description | text | |
| createdAt, updatedAt | timestamptz | |

### zones
Physical areas with capacity limits.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| eventId | uuid | FK → events, cascade delete |
| name, description | varchar | |
| capacity, currentOccupancy | integer | |
| createdAt, updatedAt | timestamptz | |

### entities
Invitation quota groups (sponsors, partners, companies).
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| eventId | uuid | FK → events, cascade delete |
| name, email, description | varchar | |
| quota | integer | invitations allowed |
| createdAt, updatedAt | timestamptz | |

### ticketTypes
Ticket type definitions.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| eventId | uuid | FK → events, cascade delete |
| name | varchar | |
| requiresSelfie, requiresEmail | boolean | |
| maxUses | integer | nullable for unlimited |
| validFrom, validUntil | timestamptz | |
| price | numeric | optional |
| createdAt, updatedAt | timestamptz | |

### zoneTicketTypes
Many-to-many junction table.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| zoneId | uuid | FK → zones, cascade delete |
| ticketTypeId | uuid | FK → ticketTypes, cascade delete |

**Unique constraint:** `(zoneId, ticketTypeId)`

### tickets
Individual tickets with QR codes.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| eventId | uuid | FK → events, cascade delete |
| ownerId | uuid | FK → owners, cascade delete |
| ticketTypeId | uuid | FK → ticketTypes, set null |
| entityId | uuid | FK → entities, set null |
| rowNumber | integer | ordering within entity |
| qrCode | varchar | unique across all tickets |
| holderName, holderEmail | varchar | |
| selfieUrl | varchar | |
| status | enum | valid/missing_requirements/used/revoked |
| isInside | boolean | currently inside venue |
| sentAt | timestamptz | when invitation sent |
| createdAt, updatedAt | timestamptz | |

### accessEvents
Audit log for access attempts.
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| ticketId | uuid | FK → tickets |
| zoneId | uuid | FK → zones |
| eventType | enum | entry/exit |
| result | enum | granted/denied |
| reason, deviceId | varchar | |
| synced | boolean | |
| createdAt | timestamptz | |

## Enums

- `member_role`: `'read' | 'write'`
- `scope_type`: `'owner' | 'event' | 'entity'`

## Key Constraints

- **Unique QR codes** across all tickets
- **One membership per (user, scope)** via composite unique index
- **Cascading deletes** maintain referential integrity:
  - Deleting owner → deletes all events, memberships, tickets
  - Deleting event → deletes zones, entities, tickets
  - Deleting user → deletes their memberships
- **Set null on delete** for optional references (venue, ticketType, entity on tickets)

## Schema Change Guidelines

- Prefer evolving existing tables over introducing redundant ones
- Maintain referential integrity (`onDelete` rules matter)
- Use `pnpm drizzle-kit push` for migrations
