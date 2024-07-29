# Airline Company Case Study

This document describes the data requirements for a fictional airline company, Anchor Air. In this case study, the company's key information requirements are identified. This information primarily deals with the assets the airline must use and manage to operate: airports, maintenance flight routes, and scheduled flights onto which
Airline Company Case Study

This document describes the data requirements for a fictional airline company, Anchor Air. In this case study, the company's key information requirements are identified. This information primarily deals with the assets the airline must use and manage to operate: airports, maintenance flight routes, and scheduled flights onto which customers book seats along with information about the passengers themselves.

---

## Employees

The company needs to keep the following information regarding its employee. In the US, all employees have a unique social security number. Other information on employees that the airline might need include facts such as the employee's name; the employee's home street address and the employees city, statement and zipcode; the employee's hone phone number; the employee's salary; and the employee's birth date. Additionally, some employees are pilot crew, while others are attendant crew. For pilots, the company need to keep the following information: the pilot's rank (probationary, junior, regular, senior); a list of the aircraft the pilot is rated to fly; should reference only aircraft the company owns; the number of flight hours the pilots has flown; the pilots home airport (this should be an airport that the airline is allowed to use); and an additional contact number for the pilot. For attendants, the company needs to keep the following information: the attendant's rank (probationary, crew, chief); the attendant's home airport (again it should be one the airline is allowed to use); and an additional contact number for the attendant

---

## Airports

An airline is only allowed to fly to specific airports. This airline is a US company and is allowed to fly between US domestic airports only. The company needs to maintain information about the airports it is allowed to use. Airports are identified in the US with a three letter code (upper case)m which are unique. Other facts about airports include which city and state they are in, and how many gates are available for boarding and debarking the aircraft customers. The number of gates must be a positive integer. All of this information is required.

---

## Airline Flight Routes

The airline has many aircraft flying every day to provide transport for its customers. By regulation, the airline is assigned certain routes between a origin airport and a destination airport. Each route (which is identified by a unique code), is schedule for the same time on the same day every week. While the origin and destination airports are the same every time the route is flown, the gates at the origin and destination airports may change from week to week. The origin and destination airports must each refer to one of the airports to which the airline flies. All of this information is required.

---

## Scheduled Flights

The airline needs to keep track of the flight routes as they are schedule each day. The schedule simply needs to track which route is being flown (see the previous section on Airline Flight Routes) on which date and what are the departure and arrival gates at the origin and destiation airport, respectively.

---

## Flight Prices

The seats available on each flight can have varying prices, depending on the class of the seat and these prices can vary from day to day. The database must track the ticket price for the following seat categories: first-class; business class; coach class; and economy class. The ticket price is for a specific seat category for a specific flight route id of the airline on a specific date. Ticket prices must be positive monetary amounts (two decimal places). All of this information is required.

---

## Passengers

The airline is required to keep certain information about passengers who have booked flights with the airline. The database includes the last name, first name, middle initial, the street address, the city, the state, the zip code, and the phone numbers of the of the passenger and the ' email. The required information for the database is the first name, the last name, the street address, the city, the state, the zip code. The the other passenger information is optional. A passenger may give a number of phone numbers or none at all.

---

## Flight Bookings

The airline must keep information that represents a passenger's booking for one of the airline's flights. The information must show which passenger is booked on a which route on which date and for what price. All of this information is required.

---

## Flight Crews

The company has to assign crew members (pilots and flight attendants) for each date that one of its routes is flown. For each occurrence of a route being flown on a particular date, the company needs to know which pilots and flight attendants are assigned for that flight on that date. There can be different numbers of crew members assigned, but assume that all flights have between two and four pilot crew and four and eight attendant crew.

Question: In the course project, you will develop a relational database to manage the data required by the airline case study. You'll design and implement a relational database for this and supply a set of operational test queries that extract information from your database. Your deliverable will be your entity-relational diagram (ERD) of your data model, the schema for your database, the SQL script that implements the tables of your database, the SQL script that populates your database with data, and the SQL script containing your operational test SQL select queries.

Please Include All Of This

---

## Data modeling design

In this step, you'll model the data requirements of your case study system. You deliverable for this step will be the

entity types and relationship types that describe your case study system. You must include an ER diagram (using the UML

format) drawn using the Dia drawing application as part of your deliverable for this step. If you have constraints that cannot

be expressed in the ER diagram, you may have an accompanying text file for your ER diagram.

In the past, some students have found it useful to have multiple ER diagrams if their model has complex relationships

that can be hard to draw in one big diagram. One strategy is to have a separate ER diagram for each entity that shows the

details for that entity (name, attributes, keys, etc) and to have one or more ER diagrams showing the relationships, with only

the entity names and keys.

---

## Relational Database Schema Development

In this step, you'll produce a set of relational database schema from your ER diagram. Your schema must identify the

the tables required with the table schema: the table attributes along with their domains, primary key constraints, alternate

key constraints, foreign key constraints, and any general column or table constraints. Be sure also to specify whether

attributes are nullable. Remember that any many-to-many relationships in your ER diagram must become tables in your

relational

In this step, you will also ensure that your tables are in 3NF.

Your deliverable for this step is an English language description of the database schema. You should include the

functional dependencies that you identified for your tables as evidence that your tables in 3NF. You are not to implement

the tables using SQL for this deliverable.

The following are suggested formats for this deliverable:

table_schema_name ( col1 : type1, ..., coln: typen)

◦you need to identify the primary and foreign key constraints

---

## Relational Database Implementation

In this step, you will implement the table schema developed in step 2, using the Postgres SQL DDL language. Your

deliverable will be a Postgres SQL DDL script which when run in Postgres creates the tables for your case study.

---

## Table Population

In this step, you'll create a data set for your database. Your deliverable will be a Postgres SQL DML script which when

run, inserts your data set into the database tables and also a text document with the data presented in a tabular format.

---

## Database Operational testing

In this final step, you'll create a Postgres SQL DML script which performs a set of queries on your populated database.

Each case study has a set of sample queries. Choose any ten of these queries and implement them. Your deliverable will be

a Postgres SQL DML script, which when run, performs the queries on your database as well as a text document with the

query results presented in a tabular form. Note, you can capture the query results from the result window of pgAdmin.

I need PDF document containing each of the five deliverables from above, table creation script for your database, table population script for your database, an operational testing script for your database.
