CREATE TABLE visits (
Visit_ID VARCHAR(20),
Visit_Date DATE,
Visit_Year INT,
Visit_Month INT,
Visit_Day VARCHAR(20),
Weekend_Flag VARCHAR(3),
Patient_ID VARCHAR(20),
Doctor_ID VARCHAR(20),
Location_ID VARCHAR(20),
Visit_Type VARCHAR(20),
Department VARCHAR(50),
Diagnosis VARCHAR(100),
Waiting_Time_Min INT,
Waiting_Time_Category VARCHAR(20),
Consultation_Duration_Min INT,
Consultation_Duration_Category VARCHAR(20),
Bill_Amount NUMERIC(15,2),
Payment_Method VARCHAR(30),
Visit_Status VARCHAR(30),
Operational_Visit INT
);

CREATE TABLE doctors (
Doctor_ID VARCHAR(10),
Doctor_Name VARCHAR(20),
Specialty	VARCHAR(50),
State	VARCHAR(20),
Region	VARCHAR(20),
Level	VARCHAR(20),
Years_Experience	INT,
Experience_Band VARCHAR(20)
);

CREATE TABLE patients(
Patient_ID	VARCHAR(20),
First_Name VARCHAR(20),
Last_Name	VARCHAR(20),
Gender	VARCHAR(7),
Age	INT,
Age_Group	VARCHAR(20),
Registration_Date	DATE,
Patient_Tenure_Years	INT,
State	VARCHAR(20),
Region	VARCHAR(20),
Patient_Category VARCHAR(20)
);

CREATE TABLE locations(
Location_ID	VARCHAR(20),
Location_Name	VARCHAR(30),
Facility_Type	VARCHAR(30),
State	VARCHAR(20),
Region	VARCHAR(20),
Area_Type VARCHAR(20)
);
SELECT * FROM visits WHERE department IS NULL;
DELETE FROM visits WHERE visit_id IS NULL; 
SELECT COUNT(*) FROM visits; 

--1. Total visits and revenue overview
SELECT 
    COUNT(visit_id) AS total_visits,
    SUM(bill_amount) AS total_revenue,
    ROUND(AVG(waiting_time_min), 1) AS avg_wait_min
FROM visits;

--2. Visits and revenue by department
SELECT department, COUNT(visit_id) AS visit_count, SUM(bill_amount) AS revenue
FROM visits GROUP BY department ORDER BY visit_count DESC;

--3. Completed visits
SELECT visit_id, patient_id, doctor_id, bill_amount
FROM visits WHERE visit_status = 'Completed';

--4. Doctor workload (JOIN, GROUP BY, alias)
SELECT d.doctor_id, d.doctor_name, d.specialty,
       COUNT(v.visit_id) AS total_visits,
       ROUND(AVG(v.waiting_time_min), 1) AS avg_wait_min
FROM visits v
JOIN doctors d ON v.doctor_id = d.doctor_id
GROUP BY d.doctor_id, d.doctor_name, d.specialty
ORDER BY total_visits DESC;

--5. Revenue by facility type and region
SELECT COALESCE(l.facility_type, 'Unmatched Location') AS facility_type,
       COALESCE(l.region, 'Unknown') AS region,
       COUNT(v.visit_id) AS visits,
       SUM(v.bill_amount) AS total_revenue
FROM visits v
LEFT JOIN locations l ON v.location_id = l.location_id
GROUP BY COALESCE(l.facility_type, 'Unmatched Location'), COALESCE(l.region, 'Unknown')
ORDER BY total_revenue DESC;

--6. Demographics by department
SELECT v.department, p.gender, COUNT(v.visit_id) AS visits, ROUND(AVG(p.age), 1) AS avg_age
FROM visits v JOIN patients p ON v.patient_id = p.patient_id
GROUP BY v.department, p.gender ORDER BY v.department, visits DESC;

--7. No-show / cancellation rate by waiting-time band
SELECT waiting_time_category,
       ROUND(100.0 * COUNT(*) FILTER (WHERE visit_status = 'No Show') / COUNT(*), 1) AS no_show_pct,
       ROUND(100.0 * COUNT(*) FILTER (WHERE visit_status = 'Cancelled') / COUNT(*), 1) AS cancel_pct
FROM visits GROUP BY waiting_time_category ORDER BY no_show_pct DESC;

--8. Doctors above the network's average visit count 
SELECT d.doctor_name, COUNT(v.visit_id) AS visit_count
FROM visits v JOIN doctors d ON v.doctor_id = d.doctor_id
GROUP BY d.doctor_name
HAVING COUNT(v.visit_id) > (
    SELECT AVG(cnt) FROM (
        SELECT doctor_id, COUNT(visit_id) AS cnt FROM visits GROUP BY doctor_id
    ) t)
ORDER BY visit_count DESC;

--9. Top 1% highest-billing visits
SELECT visit_id, patient_id, department, bill_amount
FROM visits
WHERE bill_amount >= (SELECT PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY bill_amount) FROM visits)
ORDER BY bill_amount DESC;

--10. Revenue by payment method
SELECT payment_method, COUNT(visit_id) AS transactions, SUM(bill_amount) AS total_revenue,
       ROUND(AVG(bill_amount), 2) AS avg_bill
FROM visits WHERE visit_status = 'Completed'
GROUP BY payment_method ORDER BY total_revenue DESC;

--11. Monthly trend 
SELECT visit_year, visit_month, COUNT(visit_id) AS visits, SUM(bill_amount) AS revenue
FROM visits WHERE visit_date IS NOT NULL
GROUP BY visit_year, visit_month ORDER BY visit_year, visit_month;

--12. Frequent patients (>3 visits)

SELECT p.patient_id, p.first_name || ' ' || p.last_name AS patient_name, COUNT(v.visit_id) AS visit_count
FROM visits v JOIN patients p ON v.patient_id = p.patient_id
GROUP BY p.patient_id, patient_name
HAVING COUNT(v.visit_id) > 3 ORDER BY visit_count DESC;

--13. Locations below average revenue per visit
SELECT l.location_name, l.region, ROUND(AVG(v.bill_amount), 2) AS avg_revenue_per_visit
FROM visits v JOIN locations l ON v.location_id = l.location_id
GROUP BY l.location_name, l.region
HAVING AVG(v.bill_amount) < (SELECT AVG(bill_amount) FROM visits)
ORDER BY avg_revenue_per_visit ASC;

--14. Weekend vs. weekday performance
SELECT weekend_flag, COUNT(visit_id) AS visits, ROUND(AVG(waiting_time_min), 1) AS avg_wait_min,
       SUM(bill_amount) AS revenue
FROM visits GROUP BY weekend_flag;

--15. Diagnosis frequency (top conditions)
SELECT diagnosis, COUNT(visit_id) AS occurrence_count
FROM visits
WHERE visit_status = 'Completed'
GROUP BY diagnosis
ORDER BY occurrence_count DESC
LIMIT 10;

--16. Consultation duration vs. experience band
SELECT d.experience_band, ROUND(AVG(v.consultation_duration_min), 1) AS avg_consultation_min,
       COUNT(v.visit_id) AS visits
FROM visits v JOIN doctors d ON v.doctor_id = d.doctor_id
WHERE v.visit_status = 'Completed'
GROUP BY d.experience_band
ORDER BY avg_consultation_min DESC;

--17. Visit type mix by month (seasonality of care type)
SELECT visit_month, visit_type, COUNT(visit_id) AS visit_count
FROM visits
WHERE visit_year = 2025
GROUP BY visit_month, visit_type
ORDER BY visit_month, visit_count DESC;

SELECT p.patient_category, d.specialty, l.facility_type,
       COUNT(v.visit_id) AS visits, SUM(v.bill_amount) AS revenue
FROM visits v
JOIN patients p ON v.patient_id = p.patient_id
JOIN doctors d ON v.doctor_id = d.doctor_id
JOIN locations l ON v.location_id = l.location_id
WHERE v.visit_status = 'Completed'
GROUP BY p.patient_category, d.specialty, l.facility_type
ORDER BY revenue DESC
LIMIT 15;

