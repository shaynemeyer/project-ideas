# Anchor Air - Airline Database Management System

A comprehensive relational database design project for managing a fictional US domestic airline company's operations.

## Overview

This project implements a complete database system for **Anchor Air**, handling all aspects of airline operations including flight scheduling, crew management, passenger bookings, and airport operations. The database is designed using PostgreSQL and follows proper normalization principles (3NF).

## Project Scope

The database manages:

- **Flight Operations**: Aircraft types, flight routes, scheduled flights, and multi-leg journeys
- **Personnel**: Flight crew (pilots and attendants) and ground staff with different compensation models
- **Airports**: US domestic airports with gate capacity tracking
- **Passengers**: Customer information, bookings, seat assignments, and baggage handling
- **Pricing**: Fare structures with varying seat classes and pricing rules

## Database Schema

### Main Entity Groups

**Personnel Management**
- Employees (base information)
- Flight Crew (pilots and attendants)
- Ground Staff (airport-based employees)

**Flight Operations**
- Airplane Types
- Flights (route definitions)
- Flight Instances (scheduled occurrences)
- Planned Legs (multi-leg routes)
- Leg Instances (actual leg occurrences)

**Location & Infrastructure**
- Airports (US domestic only)
- Layovers (connection points)

**Customer & Revenue**
- Passengers
- Seats (assignments)
- Baggage
- Fare Information
- Flight Fares

### Key Design Features

- **Composite Primary Keys**: Used for temporal data (Flight_Instance, Leg_Instance)
- **Multi-Leg Support**: Flights can consist of multiple legs with different crews
- **Crew Ranking System**: Hierarchical ranks for both pilots and attendants
- **US Domestic Only**: Restricted to 3-letter US airport codes
- **Gate Capacity Tracking**: Monitors available gates at each airport
- **Flexible Pricing**: Multiple fare classes per flight with varying rules

## Project Structure

```
fictional_airline/
├── airline.md           # Complete SQL schema (DDL)
├── requirements.md      # Detailed business requirements
├── airportdb-en.pdf    # Airport database reference
└── CLAUDE.md           # Development guidance
```

## Database Implementation

### Table Creation Order

Due to foreign key constraints, tables must be created in this order:

1. Independent tables: `Airplane_Type`, `Employees`, `Fare_Info`
2. Employee subtypes: `Flight_Crew`, `Ground_Staff`
3. Infrastructure: `Airports`
4. Flight definitions: `Flights`
5. Flight scheduling: `Flight_Instance`
6. Route planning: `Planned_Legs`
7. Actual operations: `Leg_Instance`
8. Customer data: `Passengers`, `Baggage`
9. Booking data: `Seats`, `Flight_Fare`, `Layovers`

### Schema Highlights

**Employees**
- Unique SSN identifier
- Home address and contact information
- Employee type (Flight Crew vs Ground Staff)

**Flight_Crew**
- Pilot ranks: Probationary → Junior → Regular → Senior
- Attendant ranks: Probationary → Crew → Chief
- Home airport assignments
- Salary-based compensation

**Ground_Staff**
- Airport-specific assignments
- Hourly wage compensation

**Airports**
- 3-letter uppercase codes (e.g., JFK, LAX, ORD)
- City and state location
- Gate capacity limits
- Assigned crew members

**Flights**
- Unique flight IDs and numbers
- Airline codes
- Operating day schedules
- Aircraft type assignments

**Flight_Instance**
- Specific flight on a specific date
- Departure and arrival times
- Origin and destination locations

**Seats**
- Individual seat assignments
- Linked to passengers and payments
- Associated with specific flight instances

**Fare_Info**
- Pricing restrictions
- Amount rules
- Multiple fare codes per flight

## Development Guidelines

### Prerequisites

- PostgreSQL database server
- SQL client (pgAdmin recommended)

### Creating the Database

1. Create tables using the DDL schema in `airline.md`
2. Follow the dependency order listed above
3. Verify all foreign key constraints are satisfied

### Populating Data

When inserting test data:

- Use valid US airport codes
- Ensure SSNs are unique
- Respect foreign key relationships
- Use realistic dates and times
- Match crew home airports when assigning flights

### Sample Queries

The database should support queries for:

- Flight schedules by date, route, or airport
- Crew availability and assignments
- Passenger bookings and itineraries
- Seat availability on flights
- Revenue calculations by flight or route
- Multi-leg journey tracking with layovers
- Airport gate utilization
- Employee work schedules and compensation

## Business Requirements

### Employees

**Pilots must track:**
- Rank level
- Aircraft certifications (must own those aircraft)
- Total flight hours
- Home airport
- Additional contact number

**Attendants must track:**
- Rank level
- Home airport
- Additional contact number

### Airports

- 3-letter codes (uppercase, unique)
- City and state
- Number of available gates (positive integer)
- All fields required

### Flight Routes

- Unique route codes
- Same schedule weekly (same day, same time)
- Fixed origin and destination airports
- Variable gate assignments

### Pricing

Four seat categories:
- First Class
- Business Class
- Coach Class
- Economy Class

Prices vary by:
- Flight route
- Date
- Seat category

### Crew Requirements

Per flight:
- 2-4 pilot crew members
- 4-8 attendant crew members

## Normalization

The schema adheres to **Third Normal Form (3NF)**:

- No repeating groups (1NF)
- All non-key attributes depend on the entire primary key (2NF)
- No transitive dependencies (3NF)

Functional dependencies should be documented for all tables when making schema changes.

## Project Deliverables

This project includes:

1. ER Diagram (UML format) showing entities and relationships
2. Relational schema with constraints and 3NF justification
3. SQL DDL script for table creation
4. SQL DML script for data population
5. SQL query script with 10+ operational test queries

## References

- `requirements.md`: Complete business case study
- `airline.md`: Full SQL schema definitions
- `airportdb-en.pdf`: Airport database structure reference

## License

This is an academic project for database design education.
