# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a database design and implementation project for a fictional airline company called **Anchor Air**. The project involves designing a relational database system to manage airline operations including flights, employees, passengers, bookings, and airport operations.

## Project Context

**Domain**: US domestic airline operations management
**Database**: PostgreSQL
**Purpose**: Academic database design project following a complete data modeling lifecycle

The project follows five phases:
1. Data modeling design (ER diagrams using UML format)
2. Relational schema development (3NF normalization)
3. Database implementation (PostgreSQL DDL scripts)
4. Table population (PostgreSQL DML scripts with sample data)
5. Operational testing (10+ test queries)

## Key Files

- `requirements.md`: Complete business requirements for Anchor Air case study
- `airline.md`: Current database schema (SQL DDL) with all table definitions
- `airportdb-en.pdf`: Reference documentation for airport database structure

## Database Architecture

### Core Entity Groups

**Personnel Management**
- `Employees`: Base employee information (SSN as unique identifier)
- `Flight_Crew`: Pilots and attendants with salary information
- `Ground_Staff`: Airport-based staff with hourly wages

**Flight Operations**
- `Airplane_Type`: Aircraft specifications (seats, manufacturer, model)
- `Flights`: Flight definitions with airline codes and operation schedules
- `Flight_Instance`: Specific flight occurrences on dates
- `Planned_Legs`: Multi-leg flight route segments
- `Leg_Instance`: Actual leg occurrences

**Airport & Location**
- `Airports`: US domestic airports (3-letter codes, city, available gates)
- `Layovers`: Connection points for multi-leg flights

**Passenger & Booking**
- `Passengers`: Customer information
- `Seats`: Seat assignments linked to flights and passengers
- `Baggage`: Passenger luggage tracking
- `Fare_Info`: Pricing rules and restrictions
- `Flight_Fare`: Fare assignments to specific flights

### Key Relationships

- Flight_Instance uses composite primary key (Flight_ID, Flight_Date)
- Leg_Instance references both Planned_Legs and Flight_Instance
- Airports are assigned Flight_Crew members
- Seats link Passengers to Flight_Instance with payment tracking

### Schema Constraints

**US-Only Operations**: All airports must be US domestic (3-letter uppercase codes)

**Employee Types**: Two distinct types (Flight_Crew vs Ground_Staff) with different compensation models

**Multi-Leg Flights**: Flights can have multiple legs with different crews per leg

**Gate Assignment**: Airport gates are limited resources tracked per airport

**Crew Requirements**:
- Pilots: Rank system (probationary → junior → regular → senior)
- Attendants: Rank system (probationary → crew → chief)
- Each crew member has home airport assignment

## Development Workflow

### Database Implementation

When creating DDL scripts:
- Use PostgreSQL-specific syntax
- All tables must include proper PRIMARY KEY constraints
- Foreign keys must reference existing tables in dependency order
- Table creation order matters due to foreign key dependencies

Recommended table creation sequence:
1. `Airplane_Type`, `Employees`, `Fare_Info` (no dependencies)
2. `Flight_Crew`, `Ground_Staff` (depend on Employees)
3. `Airports` (depends on Flight_Crew)
4. `Flights` (depends on Airplane_Type)
5. `Flight_Instance` (depends on Flights)
6. `Planned_Legs` (depends on Flights, Airports, Flight_Crew)
7. `Leg_Instance` (depends on Planned_Legs, Flight_Instance)
8. `Passengers`, `Baggage` (Baggage depends on Passengers)
9. `Seats` (depends on Flight_Instance, Passengers)
10. `Flight_Fare` (depends on Flight_Instance, Fare_Info)
11. `Layovers` (depends on Flight_Instance)

### Data Population

When creating DML scripts:
- Insert data respecting foreign key constraints (same order as table creation)
- Use realistic US airport codes (e.g., JFK, LAX, ORD, DFW, ATL)
- Employee SSNs must be unique
- Flight dates should be realistic and consistent
- Ensure crew assignments match home airports where applicable

### Query Development

Sample operational queries should demonstrate:
- Flight schedules by route/date/airport
- Crew assignments and availability
- Passenger bookings and seat assignments
- Revenue calculations using fare information
- Multi-leg journey tracking
- Airport capacity and gate utilization
- Employee work schedules

## Important Design Notes

**Normalization**: Schema must be in 3rd Normal Form (3NF). Document functional dependencies when making changes.

**Temporal Data**: The system tracks both planned schedules (Planned_Legs, Flights) and actual instances (Flight_Instance, Leg_Instance).

**Composite Keys**: Multiple tables use composite primary keys (Flight_Instance, Leg_Instance, Planned_Legs, Flight_Fare, Layovers).

**Data Integrity**: The schema enforces referential integrity through foreign keys. Ensure all references are valid before insertion.

## Common Queries to Test

- Find all flights between two airports on a specific date
- List available seats on a flight
- Calculate total revenue for a flight
- Find crew assignments for a specific flight
- Identify passengers with baggage
- Track multi-leg journeys with layovers
- List all flights an employee is assigned to
- Find airports with available gate capacity
