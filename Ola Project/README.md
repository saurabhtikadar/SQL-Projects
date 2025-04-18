<p align="center">
  <img src="https://github.com/user-attachments/assets/2f7cf2f2-96fb-44a5-b3e0-3989b5e38aad" alt="Ola Project - SQL Data Analyst" width="300">
</p>

<h1 align="center">Ola Project - SQL Data Analyst</h1>


This is an SQL-based project aimed at solving data analysis challenges for Ola, a popular ride-hailing service. The solution is being implemented using **PostgreSQL** as the database management system.

## 🛠️Project Overview
In this project, I will be analyzing various datasets related to Ola's operations, including ride details, customer ratings, driver performance, cancellations, and payment methods. The project will involve performing advanced SQL queries to extract insights, generate reports, and create views that can provide meaningful data for decision-making.

  
## 🤖Technology Use
- **Database**: PostgreSQL
- **Language**: SQL (Structured Query Language)
  

## **💾Dataset Preparation in PostgreSQL**
To analyze the data and address the Bussiness Question of this project, 
we need to create a table in PostgreSQL that includes all the required columns.

**1. 🛢️Use a Staging Table with All Columns as `TEXT`:**

Create a temporary staging table where all columns are defined as `TEXT` to ensure the CSV data is imported `without errors`.
```sql
CREATE TABLE bookings (
    date TEXT,
    time TEXT,
    booking_id TEXT,
    booking_status TEXT,
    customer_id TEXT,
    vehicle_type TEXT,
    pickup_location TEXT,
    drop_location TEXT,
    v_tat TEXT,
    c_tat TEXT,
    canceled_rides_by_customer TEXT,
    canceled_rides_by_driver TEXT,
    incomplete_rides TEXT,
    incomplete_rides_reason TEXT,
    booking_value TEXT,
    payment_method TEXT,
    ride_distance TEXT,
    driver_ratings TEXT,
    customer_rating TEXT);

```
**2. 📥Import the [Data](https://github.com/saurabhtikadar/SQL-Projects/blob/main/Ola%20Project/Bookings.csv) into the Staging Table:**

Use the `COPY` command to load the data into the staging table.
```sql
COPY bookings
FROM '/path/to/your/Sample.csv'
DELIMITER ','
CSV HEADER;
```
**3. 📇Extract the Date from the `Date Column`**

If the `date_column` in your [bookings.csv](https://github.com/saurabhtikadar/SQL-Projects/blob/main/Ola%20Project/Bookings.csv) file contains both date and time (e.g., `"26-07-2024 14:00"`), and you want to change its data type to `DATE`,
you will face an issue because the `DATE` data type cannot store time information. To handle this:
```sql
UPDATE bookings
SET date = LEFT(date, 10);
```
**Note :** This query extracts the first 10 characters from the `date_column` (which correspond to the date in the format `DD-MM-YYYY`) and updates the column.

**4. 🔄Change the Data Type to `DATE`**

After ensuring the column contains only the date, change its data type to `DATE` using:
```sql
ALTER TABLE bookings
ALTER COLUMN date TYPE DATE
USING TO_DATE(date, 'DD-MM-YYYY');
```
**Pro Tip :** To avoid these issues altogether, you can clean and prepare your data in `Excel` before importing it into PostgreSQL.

**5. ➡️Replace `"null"` with SQL `NULL`**

To ensure a smooth data type conversion, first check if the column contains string representations of null values (e.g., `"null"`) and convert them to SQL `NULL`.
This ensures that non-numeric values won't cause `errors` during the conversion process.

**Example :** In this [Sample.csv] file presence of `null` values in the `driver_ratings` and `customer_rating` columns:
```sql
UPDATE bookings
SET driver_ratings = NULL, customer_rating = NULL
WHERE driver_ratings = 'null' OR customer_rating = 'null';

```
**6. 🔄Convert `NUMERIC` Data Type**

Now, Any columns that store digits but have their data type as `TEXT` will be converted to a `NUMERIC` data type for better accuracy and efficiency.
```sql
ALTER TABLE bookings
ALTER COLUMN driver_ratings TYPE NUMERIC USING driver_ratings::NUMERIC,
ALTER COLUMN customer_rating TYPE NUMERIC USING customer_rating::NUMERIC,
ALTER COLUMN ride_distance TYPE  NUMERIC USING ride_distance::NUMERIC,
ALTER COLUMN booking_value TYPE  NUMERIC USING booking_value::NUMERIC;
```

<h1 align="center">📊 Data Analysis with SQL Queries</h1>

Below are the steps to solve each business question. Simply run the query `SELECT * FROM view_name;` to see the results for each problem.

**1. 🎯Retrieve all successful bookings**: The query retrieves all bookings that were successfully completed.
```sql
CREATE OR REPLACE VIEW success_booking AS 
SELECT  * FROM bookings
WHERE booking_status = 'Success';
```
**View Name :** `success_booking`

**Description :** Lists all bookings that were successfully completed.
```sql
SELECT * FROM success_booking;
```
Result : ![Image](https://github.com/user-attachments/assets/7e71d4e6-4a20-4258-8861-ab6b0d77033b)

**2. 🚗Find the average ride distance for each vehicle type**: The query calculates the average ride distance for each type of vehicle.
```sql
CREATE OR REPLACE VIEW avg_dis_for_each_vehicle AS
SELECT vehicle_type,ROUND(AVG(ride_distance),2) AS avg_distance
FROM bookings
GROUP BY vehicle_type ORDER BY vehicle_type;
```
**View Name :** `avg_dis_for_each_vehicle`

**Description :** Shows the average ride distance for each type of vehicle.
```sql
SELECT * FROM avg_dis_for_each_vehicle;
```
Result :

![Image](https://github.com/user-attachments/assets/3706295f-bb4b-4ee2-820e-ea839ff73577)

**3. ❌Get the total number of cancelled rides by customers**: The query counts the number of rides that were cancelled by customers.
```sql
CREATE OR REPLACE VIEW total_cancelled_rides_by_customers AS
SELECT COUNT(*) AS total_cancelled_rides_by_customers
FROM bookings
WHERE booking_status ='Canceled by Customer';
```
**View Name :** `total_cancelled_rides_by_customers`

**Description :** Displays the total number of rides cancelled by customers.
```sql
SELECT * FROM total_cancelled_rides_by_customers;
```
Result :

![Image](https://github.com/user-attachments/assets/664edf17-652b-4351-8feb-8d4d529c7b7e)

**4. 🏆List the top 5 customers who booked the highest number of rides**: The query identifies the top 5 customers with the most bookings.
```sql
CREATE OR REPLACE VIEW top_5_customers_high_booking AS
SELECT customer_id,COUNT(booking_id) AS total_booking
FROM bookings
GROUP BY customer_id order by total_booking DESC LIMIT 5;
```
**View Name :** `top_5_customers_high_booking`

**Description :** Identifies the top 5 customers based on the number of bookings made.
```sql
SELECT  * FROM top_5_customers_high_booking;
```
Result : 

![Image](https://github.com/user-attachments/assets/bfcbd47e-34a1-4654-80ea-5277960177ac)

**5. ❌Get the number of rides cancelled by drivers due to personal and car-related issues**: The query counts rides cancelled by drivers due to `personal` or `car-related` issues.
```sql
CREATE OR REPLACE VIEW cancelled_by_drivers_due_to_personal_and_car_related_issues AS
SELECT COUNT(*) AS cancelled_by_drivers_due_to_personal_and_car_related_issues
FROM bookings
WHERE canceled_rides_by_driver ='Personal & Car related issue';
```
**View Name :** `cancelled_by_drivers_due_to_personal_and_car_related_issues`

**Description :** Shows the count of rides cancelled by drivers for personal or car-related issues.
```sql
SELECT * FROM cancelled_by_drivers_due_to_personal_and_car_related_issues;
```
Result : 

![Image](https://github.com/user-attachments/assets/66456c71-454e-49dc-b297-8c3f2333b0f2)

**6. ⭐Find the maximum and minimum driver ratings for Prime Sedan bookings**: The query identifies the highest and lowest driver ratings for `"Prime Sedan"` bookings.
```sql
CREATE OR REPLACE VIEW maximum_and_minimum_driver_ratings_for_Prime_Sedan AS
SELECT vehicle_type,
    MAX(driver_ratings) AS Max_d_ratings,
    MIN(driver_ratings) AS Min_d_ratings
FROM bookings
WHERE vehicle_type = 'Prime Sedan' AND driver_ratings IS NOT NULL
GROUP BY vehicle_type;
```
If you had skipped steps **[`5 and 6 of data preparation`]()**, you could still achieve the desired results using the SQL query below:
```sql
CREATE OR REPLACE VIEW maximum_and_minimum_driver_ratings_for_Prime_Sedan AS
SELECT vehicle_type,
    MAX(driver_ratings) AS Max_d_ratings,
    MIN(driver_ratings) AS Min_d_ratings
FROM bookings
WHERE vehicle_type = 'Prime Sedan' AND driver_ratings != 'null'
GROUP BY vehicle_type;
```
**View Name :** `maximum_and_minimum_driver_ratings_for_Prime_Sedan`

**Description :** Retrieves the highest and lowest driver ratings for "Prime Sedan" bookings.
```sql
SELECT * FROM maximum_and_minimum_driver_ratings_for_Prime_Sedan;
```
Result :

![Image](https://github.com/user-attachments/assets/435297e9-52a5-4d2a-a3e4-6323c312415e)

**7. 💳Retrieve all rides where payment was made using UPI**: The query lists all rides where the payment method was `UPI`.
```sql
CREATE OR REPLACE VIEW use_upi AS
SELECT * FROM bookings
WHERE payment_method = 'UPI';
```
**View Name :** `use_upi`

**Description :** Lists all rides where the payment method was UPI.
```sql
SELECT * FROM use_upi;
```
Result :

![Image](https://github.com/user-attachments/assets/c13366fb-91e1-418a-bdf7-162ad9302334)

**8. 🚜Find the average customer rating per vehicle type**: The query calculates the average customer rating for each type of vehicle.
```sql
CREATE OR REPLACE VIEW avg_cus_rat_per_vehicle_type AS
SELECT vehicle_type,ROUND(AVG(customer_rating),2) AS avg_cus_rating
FROM bookings
GROUP BY vehicle_type;
```
**View Name :** `avg_cus_rat_per_vehicle_type`

**Description :** Displays the average customer rating for each vehicle type.
```sql
SELECT * FROM avg_cus_rat_per_vehicle_type;
```
Result :

![Image](https://github.com/user-attachments/assets/b66096ca-18cb-4f45-94b5-24d98d398ecf)

**9. 📝Calculate the total booking value of rides completed successfully**: The query sums up the booking values of successfully completed rides.
```sql
CREATE OR REPLACE VIEW total_suc_value AS
SELECT booking_status,SUM(booking_value) AS total_value
FROM bookings
GROUP BY booking_status
HAVING booking_status = 'Success';
```
**View Name :** `total_suc_value`

**Description :** Shows the total value of bookings that were successfully completed.
```sql
SELECT * FROM total_suc_value;
```
Result :

![Image](https://github.com/user-attachments/assets/b7978618-1f89-4ba9-a2b0-9f1bd273d93f)

**10. 📚List all incomplete rides along with the reason**: The query retrieves incomplete rides and their corresponding reasons.
```sql
CREATE OR REPLACE VIEW inc_ride_with_reason AS
SELECT booking_id,incomplete_rides_reason
FROM bookings
WHERE incomplete_rides_reason != 'null';
```
**View Name :** `inc_ride_with_reason`

**Description :** Retrieves incomplete rides and their respective reasons.
```sql
SELECT * FROM inc_ride_with_reason;
```
Result : 

![Image](https://github.com/user-attachments/assets/e25de37c-0efe-4044-a48c-5ffec09b8d4d)

## 🚨Future Work
- Optimizing queries for performance
- Integrating additional datasets to further enhance the analysis
- Building interactive dashboards using the results of the queries

## 🛠️How to Run
1. Clone the repository.
2. Set up the PostgreSQL database.
3. Run the SQL scripts to initialize the schema and data.
4. Execute the queries and views to generate reports.

## ⚠️Conclusion
This project will serve as a comprehensive SQL-based analysis tool for Ola, helping to improve operations and provide deeper insights into their service.

---

Feel free to explore the repository and contribute to this ongoing analysis!

### 📜Explanation of the Added SQL Queries:
- **Query 1-10**: Each query directly addresses a specific question, such as retrieving successful bookings, calculating average ride distances, or listing the top customers. They are written in standard SQL and designed to work with the "bookings" table.

Aap is template ko apne project ki specifics ke hisaab se modify kar sakte hain.

