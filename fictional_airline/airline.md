# Airline Datastructure

```sql
CREATE TABLE Airplane_Type (
    Type_ID int NOT NULL,
    Num_Seats int NOT NULL,
    Manufacturer varchar(255) NOT NULL,
    Model_Type varchar(255) NOT NULL,
    PRIMARY KEY (Type_ID)
);

CREATE TABLE Flights (
    Flight_ID varchar(255) NOT NULL,
    Flight_Number int NOT NULL,
    Airline_Code varchar(255) NOT NULL,
    Operation_Days varchar(255) NOT NULL,
    Plane_Type_ID int NOT NULL,
    PRIMARY KEY (Flight_ID),
 FOREIGN KEY (Plane_Type_ID) REFERENCES Airplane_Type(Type_ID)
);

CREATE TABLE Flight_Instance (
    Flight_ID varchar(255) NOT NULL,
    Flight_Date DATE NOT NULL,
    Start_Location varchar(255) NOT NULL,
    Destination_Location varchar (255) NOT NULL,
    Departure_Time TIMESTAMP NOT NULL,
    Arrival_Time TIMESTAMP NOT NULL,
    PRIMARY KEY (Flight_ID, Flight_Date),
    FOREIGN KEY (Flight_ID) REFERENCES Flights(Flight_ID)
);

CREATE TABLE Employees (
    Emp_ID varchar(255) NOT NULL,
    Fname varchar(255) NOT NULL,
    Lname varchar(255) NOT NULL,
    Phone varchar(255),
    Email varchar(255),
    Address varchar(255),
    Emp_Type varchar(255),
    PRIMARY KEY (Emp_ID)
);

CREATE TABLE Passengers (
    Passenger_ID varchar(255) NOT NULL,
    Fname varchar(255) NOT NULL,
    Lname varchar(255) NOT NULL,
    Phone varchar(255),
    Email varchar(255),
    Address varchar(255),
    PRIMARY KEY (Passenger_ID)
);



CREATE TABLE Flight_Crew (
    Emp_ID varchar(255) NOT NULL,
    Salary int NOT NULL,
    PRIMARY KEY (Emp_ID),
    FOREIGN KEY (Emp_ID) REFERENCES Employees(Emp_ID)
);

CREATE TABLE Airports (
    Airport_ID varchar(255) NOT NULL,
    Airport_Name varchar(255) NOT NULL,
    City varchar(255) NOT NULL,
    Crew_ID varchar(255) NOT NULL,
    PRIMARY KEY (Airport_ID),
    FOREIGN KEY (Crew_ID) REFERENCES Flight_Crew(Emp_ID)
);

CREATE TABLE Planned_Legs(
    Flight_ID varchar(255) NOT NULL,
    Leg_Number int NOT NULL,
    Departure_Airport_ID varchar(255) NOT NULL,
    Destination_Airport_ID varchar (255) NOT NULL,
    Departure_Time TIMESTAMP NOT NULL,
    Arrival_Time TIMESTAMP NOT NULL,
 Crew_ID varchar(255) NOT NULL,
    PRIMARY KEY (Flight_ID, Leg_Number),
    FOREIGN KEY (Departure_Airport_ID) REFERENCES Airports(Airport_ID),
    FOREIGN KEY (Destination_Airport_ID) REFERENCES Airports(Airport_ID),
    FOREIGN KEY (Crew_ID) REFERENCES Flight_Crew(Emp_ID)
);

CREATE TABLE Leg_Instance (
    Flight_ID varchar(255) NOT NULL,
    Leg_Number int NOT NULL,
    Leg_Date TIMESTAMP NOT NULL,
    Departure_Time TIMESTAMP NOT NULL,
    Arrival_Time TIMESTAMP NOT NULL,
    PRIMARY KEY (Flight_ID, Leg_Number, Leg_Date),
    FOREIGN KEY (Flight_ID, Leg_Number) REFERENCES Planned_Legs(Flight_ID, Leg_Number),
 FOREIGN KEY (Flight_ID, Leg_Date) REFERENCES Flight_Instance (Flight_ID, Flight_Date)
);


CREATE TABLE Baggage (
    Code varchar(255) NOT NULL,
    Bag_Count int NOT NULL,
    Origin varchar(255) NOT NULL,
    Destination varchar(255) NOT NULL,
    Passenger_ID varchar(255) NOT NULL,
    PRIMARY KEY (Code),
    FOREIGN KEY (Passenger_ID) REFERENCES Passengers(Passenger_ID)
);

CREATE TABLE Fare_Info (
    Code varchar(255) NOT NULL,
    Restrictions varchar(255),
    Amount_Rule varchar(255),
    PRIMARY KEY (Code)
);

CREATE TABLE Seats (
    Seat_Number varchar(255) NOT NULL,
    Flight_ID varchar(255) NOT NULL,
    Flight_Date DATE NOT NULL,
    Payment_ID varchar(255) NOT NULL,
    Passenger_ID varchar(255),
    PRIMARY KEY (Seat_Number),
    FOREIGN KEY (Flight_ID,Flight_Date) REFERENCES Flight_Instance(Flight_ID,Flight_Date),
    FOREIGN KEY (Passenger_ID) REFERENCES Passengers(Passenger_ID)
);


CREATE TABLE Flight_Fare (
    Flight_ID varchar(255) NOT NULL,
    Flight_Date DATE NOT NULL,
    Fare_Code varchar(255) NOT NULL,
    PRIMARY KEY (Flight_ID, Flight_Date, Fare_Code),
    FOREIGN KEY (Flight_ID, Flight_Date) REFERENCES Flight_Instance(Flight_ID, Flight_Date),
    FOREIGN KEY (Fare_Code) REFERENCES Fare_Info(Code)
);




CREATE TABLE Layovers (
    Flight_ID varchar(255) NOT NULL,
    Flight_Date DATE NOT NULL,
    Layover varchar(255) NOT NULL,
    PRIMARY KEY (Flight_ID, Flight_Date, Layover),
    FOREIGN KEY (Flight_ID, Flight_Date) REFERENCES Flight_Instance(Flight_ID, Flight_Date)
);

CREATE TABLE Ground_Staff (
    Emp_ID varchar(255) NOT NULL,
    Hourly_Wage int NOT NULL,
    Airport_ID varchar(255) NOT NULL,
    PRIMARY KEY (Emp_ID),
    FOREIGN KEY (Emp_ID) REFERENCES Employees(Emp_ID),
    FOREIGN KEY (Airport_ID) REFERENCES Airports(Airport_ID)
);
```
