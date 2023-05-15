# Part 1:

## A) Implementing Object-Relational version of a subset of the NorthEastNinjas Database:

### 1. Designing the logical model:
```
-- Create object types
CREATE TYPE AddressType AS OBJECT (
  street VARCHAR2(50),
  city VARCHAR2(50),
  state VARCHAR2(50),
  zip VARCHAR2(10)
);

CREATE TYPE NinjaType AS OBJECT (
  ninja_id NUMBER,
  name VARCHAR2(50),
  address AddressType,
  weapons VARCHAR2(200),
  PRIMARY KEY (ninja_id)
);

CREATE TYPE MissionType AS OBJECT (
  mission_id NUMBER,
  name VARCHAR2(50),
  location AddressType,
  PRIMARY KEY (mission_id)
);

CREATE TYPE AssignmentType AS OBJECT (
  ninja_id NUMBER,
  mission_id NUMBER,
  status VARCHAR2(20),
  PRIMARY KEY (ninja_id, mission_id),
  FOREIGN KEY (ninja_id) REFERENCES NinjaType(ninja_id),
  FOREIGN KEY (mission_id) REFERENCES MissionType(mission_id)
);
```

### 2. Implementing the logical design:

```
-- Create tables
CREATE TABLE Ninjas (
  ninja_obj NinjaType
) NESTED TABLE ninja_obj.address STORE AS Ninja_AddressTab;

CREATE TABLE Missions (
  mission_obj MissionType
) NESTED TABLE mission_obj.location STORE AS Mission_LocationTab;

CREATE TABLE Assignments (
  assignment_obj AssignmentType
);

-- Establish relationships
ALTER TABLE Assignments ADD CONSTRAINT fk_ninja_assignment
  FOREIGN KEY (ninja_id) REFERENCES Ninjas (ninja_obj.ninja_id);

ALTER TABLE Assignments ADD CONSTRAINT fk_mission_assignment
  FOREIGN KEY (mission_id) REFERENCES Missions (mission_obj.mission_id);

```

## B) Populating the OR version of the subset of the NorthEastNinjas Database:
```
-- Insert sample data
INSERT INTO Ninjas VALUES (NinjaType(1, 'Ninja 1', AddressType('Street 1', 'City 1', 'State 1', '12345'), 'Katana'));
INSERT INTO Ninjas VALUES (NinjaType(2, 'Ninja 2', AddressType('Street 2', 'City 2', 'State 2', '23456'), 'Shuriken'));
INSERT INTO Missions VALUES (MissionType(1, 'Mission 1', AddressType('Street 3', 'City 3', 'State 3', '34567')));
INSERT INTO Missions VALUES (MissionType(2, 'Mission 2', AddressType('Street 4', 'City 4', 'State 4', '45678')));
INSERT INTO Assignments VALUES (AssignmentType(1, 1, 'Active'));
INSERT INTO Assignments VALUES (AssignmentType(2, 2, 'Completed'));
```

## C) Writing Queries on the OR version of the subset of the NorthEastNinjas Database:
```
-- Query 1: List all ninjas and their assigned missions
SELECT n.ninja_obj.name, m.mission_obj.name
FROM Ninjas n
JOIN Assignments a ON n.ninja_obj.ninja_id = a.assignment_obj.ninja_id
JOIN Missions m ON a.assignment_obj.mission_id = m.mission_obj.mission_id;

-- Query 2: Count the number of active assignments for each ninja
SELECT n.ninja_obj.name, COUNT(a.assignment_obj.ninja_id) AS active_assignments
FROM Ninjas n
LEFT JOIN Assignments a ON n.ninja_obj.ninja_id = a.assignment_obj.ninja_id AND a.assignment_obj.status = 'Active'
GROUP BY n.ninja_obj.name;

-- Query 3: Find the ninjas who are not assigned to any mission
SELECT n.ninja_obj.name
FROM Ninjas n
LEFT JOIN Assignments a ON n.ninja_obj.ninja_id = a.assignment_obj.ninja_id
WHERE a.assignment_obj.ninja_id IS NULL;

-- Query 4: Find the missions with ninjas assigned to them
SELECT m.mission_obj.name
FROM Missions m
JOIN Assignments a ON m.mission_obj.mission_id = a.assignment_obj.mission_id;

-- Query 5: Find the missions with no assigned ninjas
SELECT m.mission_obj.name
FROM Missions m
LEFT JOIN Assignments a ON m.mission_obj.mission_id = a.assignment_obj.mission_id
WHERE a.assignment_obj.mission_id IS NULL;
```