  Fleet Maintenance & Work Order Processing (PL/SQL Project)
Overview

This PL/SQL project demonstrates how to automate vehicle maintenance processing using advanced Oracle PL/SQL features.
The system evaluates work orders, calculates part and labor costs, and generates a structured summary using:

Collections (Indexed Tables)

Record Types

Cursors

GOTO Statements

Joins & Relational Tables

The design mirrors real-world fleet maintenance operations used in transport companies, logistics firms, and government fleets.

  Problem Description

A transport company records vehicle maintenance operations (parts replaced, labor hours, technician rates).
To improve reporting and reduce manual errors, they want a PL/SQL system that:

Retrieves maintenance information from relational tables

Stores part costs in memory using collections

Stores maintenance summary using records

Detects missing parts using GOTO blocks

Calculates:

Total parts cost

Total labor cost

Full maintenance cost

Outputs a formatted Work Order Summary

  Database Schema
vehicles
Column	Type	Description
vehicle_id	NUMBER (PK)	Unique vehicle
plate_number	VARCHAR2(20)	License plate
maintenance
Column	Type	Description
maintenance_id	NUMBER PK	Maintenance record
vehicle_id	NUMBER FK	Vehicle
labor_hours	NUMBER	Hours worked
labor_rate	NUMBER	Rate per hour
created_at	DATE	When maintenance was done
parts
Column	Type	Description
part_id	NUMBER PK	Part identifier
part_name	VARCHAR2(50)	Name
unit_price	NUMBER	Cost
maintenance_parts
Column	Type	Description
maintenance_id	NUMBER FK	Links maintenance
part_id	NUMBER FK	Links part
  Example Sample Data
INSERT INTO vehicles VALUES (1, 'RAD-123Z');

INSERT INTO maintenance VALUES (100, 1, 3.5, 45, SYSDATE);

INSERT INTO parts VALUES (1, 'Engine Oil', 250);
INSERT INTO parts VALUES (3, 'Brake Pads', 300);
INSERT INTO parts VALUES (4, 'Air Filter', 300);

INSERT INTO maintenance_parts VALUES (100, 1);
INSERT INTO maintenance_parts VALUES (100, 3);
INSERT INTO maintenance_parts VALUES (100, 4);

COMMIT;

  PL/SQL Core Concepts Used
Feature	Purpose
Record Types	Hold structured maintenance summary
Collections	Store part costs during runtime
Cursor	Fetch dynamic maintenance rows
GOTO	Skip missing part data gracefully
DBMS_OUTPUT	Print formatted summary
  Main PL/SQL Procedure (Generates Work Order Summary)

✔ This is the missing part you asked for — the exact procedure that produces your sample output.

CREATE OR REPLACE PROCEDURE show_work_order_summary(
    p_plate IN VARCHAR2
) AS
    -- Record type for summary
    TYPE maintenance_summary_rec IS RECORD(
        v_plate        VARCHAR2(20),
        v_labor_hours  NUMBER,
        v_labor_rate   NUMBER,
        v_part_ids     VARCHAR2(200),
        v_part_cost    NUMBER
    );

    v_summary maintenance_summary_rec;

    -- Indexed table for part costs
    TYPE part_costs_tbl IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
    v_part_costs   part_costs_tbl;

    -- Cursor to fetch maintenance part info
    CURSOR c_parts IS
        SELECT mp.part_id, p.unit_price
        FROM maintenance_parts mp
        JOIN parts p ON mp.part_id = p.part_id
        WHERE mp.maintenance_id = (
            SELECT maintenance_id
            FROM maintenance
            WHERE vehicle_id = (
                SELECT vehicle_id FROM vehicles WHERE plate_number = p_plate
            )
            ORDER BY created_at DESC
            FETCH FIRST 1 ROW ONLY
        );

    v_total_parts_cost NUMBER := 0;
    v_index NUMBER := 0;

BEGIN
    -- Fetch summary info
    SELECT plate_number, labor_hours, labor_rate
    INTO v_summary.v_plate, v_summary.v_labor_hours, v_summary.v_labor_rate
    FROM vehicles v
    JOIN maintenance m ON m.vehicle_id = v.vehicle_id
    WHERE v.plate_number = p_plate
    ORDER BY m.created_at DESC
    FETCH FIRST 1 ROW ONLY;

    v_summary.v_part_ids := '';

    -- Load part info into collection
    FOR part_row IN c_parts LOOP
        v_index := v_index + 1;
        v_part_costs(v_index) := part_row.unit_price;
        v_summary.v_part_ids := v_summary.v_part_ids || part_row.part_id || ', ';
    END LOOP;

    -- No parts used? Jump using GOTO
    IF v_index = 0 THEN
        GOTO no_parts;
    END IF;

    -- Compute total part cost
    FOR i IN 1..v_index LOOP
        v_total_parts_cost := v_total_parts_cost + v_part_costs(i);
    END LOOP;

    v_summary.v_part_cost := v_total_parts_cost;

    -- Display output
    DBMS_OUTPUT.PUT_LINE('Work Order Summary');
    DBMS_OUTPUT.PUT_LINE('-------------------------');
    DBMS_OUTPUT.PUT_LINE('Vehicle Plate       : ' || v_summary.v_plate);
    DBMS_OUTPUT.PUT_LINE('Parts Used (IDs)    : ' || RTRIM(v_summary.v_part_ids, ', '));
    DBMS_OUTPUT.PUT_LINE('Total Parts Cost    : ' || v_summary.v_part_cost);
    DBMS_OUTPUT.PUT_LINE('Labor Hours         : ' || v_summary.v_labor_hours);
    DBMS_OUTPUT.PUT_LINE('Labor Rate          : ' || v_summary.v_labor_rate);
    DBMS_OUTPUT.PUT_LINE('Total Labor Cost    : ' || 
        (v_summary.v_labor_hours * v_summary.v_labor_rate));
    DBMS_OUTPUT.PUT_LINE('Overall Total Cost  : ' ||
        (v_summary.v_part_cost + 
        (v_summary.v_labor_hours * v_summary.v_labor_rate)));
    RETURN;

    -- Skip block
    <<no_parts>>
        DBMS_OUTPUT.PUT_LINE('No parts used for this maintenance record.');
END;
/

▶ Running the Procedure
BEGIN
    show_work_order_summary('RAD-123Z');
END;
/

   Example Output (Matches Your Sample Exactly)
Work Order Summary  
-------------------------
Vehicle Plate       : RAD-123Z
Parts Used (IDs)    : 1, 3, 4
Total Parts Cost    : 850
Labor Hours         : 3.5
Labor Rate          : 45
Total Labor Cost    : 157.5
Overall Total Cost  : 1007.5

   Author

Student: ASDODJI Le Sage (27232)
Course: Database development with PL/SQL
Instructor: Eric Maniraguha
Date: November 2025
