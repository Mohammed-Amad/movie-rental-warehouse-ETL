# Movie Rental Data Warehouse ETL
A data warehouse design and ETL pipeline for a movie rental OLTP database using MySQL, Python, Pandas, SQLAlchemy, and Matplotlib.

## 1. Introduction

This project designs and implements a data warehouse for a movie rental OLTP database.  
The source database supports daily operations such as managing customers, films, inventory, rentals, payments, staff, stores, and locations.

Although the OLTP schema is suitable for transaction processing, it is not optimized for analytical reporting. Therefore, this project reorganizes the operational data into a star schema using dimensional modeling principles. The warehouse supports analysis of rental activity, revenue, film popularity, customer behavior, store performance, staff performance, location activity, and time-based trends.

- **Source Database:** MySQL `movie_rental_oltp`
- **Warehouse Database:** MySQL `movie_rental_dw`
- **Tools:** Python, Pandas, SQLAlchemy, PyMySQL, Matplotlib
- **Schema Type:** Star Schema

---

## 2. Business Questions

The data warehouse is designed to answer the following business questions:

1. Which films are rented most frequently?
2. Which films generate the highest revenue?
3. Which film categories are most popular?
4. Which stores generate the highest number of rentals?
5. Which stores generate the highest revenue?
6. Which customers rent the most films?
7. Which customers generate the highest revenue?
8. How does rental activity change over time?
9. How does revenue change by month, quarter, or year?
10. Which staff members process the highest number of rentals or payments?
11. Which cities or countries generate the highest customer activity?
12. What is the average rental duration for different film categories?
13. Which films are returned late most often?
14. How does store performance differ by location?
15. What are the most active customer locations?

These questions help business managers make decisions about inventory, revenue, customers, stores, staff, and market locations.

---

## 3. Dimensional Model Design

### 3.1 Selected Business Processes

The warehouse focuses on two main business processes:

1. **Rental Transactions**
2. **Payment Transactions**

These processes were selected because they represent the main measurable activities in the movie rental business.

### 3.2 Fact Tables

#### `fact_rental`

One row represents one rental transaction.

**Measures:**

- `rental_count`
- `actual_rental_duration_days`
- `expected_rental_duration_days`
- `is_late_return`
- `late_days`

**Foreign Keys:**

- `rental_date_key`
- `return_date_key`
- `customer_key`
- `film_key`
- `store_key`
- `staff_key`

#### `fact_payment`

One row represents one payment transaction.

**Measures:**

- `payment_count`
- `payment_amount`
- `revenue`

**Foreign Keys:**

- `payment_date_key`
- `customer_key`
- `film_key`
- `store_key`
- `staff_key`

### 3.3 Dimension Tables

#### `dim_date`

Generated from rental, return, and payment dates.

**Main attributes:**

- `date_key`
- `full_date`
- `day`
- `month`
- `month_name`
- `quarter`
- `year`
- `weekday_name`
- `is_weekend`

#### `dim_customer`

Built from `customer`, `address`, `city`, and `country`.

**Main attributes:**

- `customer_key`
- `customer_id`
- `full_name`
- `email`
- `active_status`
- `address`
- `city`
- `country`
- `create_date`

#### `dim_film`

Built from `film`, `language`, `film_category`, and `category`.

**Main attributes:**

- `film_key`
- `film_id`
- `title`
- `description`
- `release_year`
- `rental_duration`
- `rental_rate`
- `replacement_cost`
- `rating`
- `language_name`
- `category_name`

#### `dim_store`

Built from `store`, `address`, `city`, and `country`.

**Main attributes:**

- `store_key`
- `store_id`
- `manager_staff_id`
- `store_address`
- `city`
- `country`

#### `dim_staff`

Built from `staff`, `address`, `city`, and `country`.

**Main attributes:**

- `staff_key`
- `staff_id`
- `full_name`
- `email`
- `active_status`
- `store_id`
- `city`
- `country`

### Shared Dimensions

Both fact tables share the following dimensions:

- `dim_date`
- `dim_customer`
- `dim_film`
- `dim_store`
- `dim_staff`

---

## 4. Dimensional Model Diagram

```text
                        dim_date
                           |
dim_customer ------- fact_rental ------- dim_film
                           |
                       dim_store
                           |
                       dim_staff


                        dim_date
                           |
dim_customer ------- fact_payment ------ dim_film
                           |
                       dim_store
                           |
                       dim_staff
```
## 5. ETL Design

### 5.1 Extract

The ETL process extracts the required OLTP tables from MySQL:

- `rental`
- `payment`
- `customer`
- `film`
- `inventory`
- `store`
- `staff`
- `address`
- `city`
- `country`
- `category`
- `film_category`
- `language`
- `actor`
- `film_actor`

These tables are needed because they contain the operational data required to build the warehouse dimensions and fact tables. For example, `rental` and `payment` are used to build the fact tables, while `customer`, `film`, `store`, `staff`, and location-related tables are used to build the dimensions.

### 5.2 Transform

The transformation phase cleans and reshapes the OLTP data into analytical structures.

Main transformations include:

- Joining related OLTP tables to build meaningful dimensions.
- Combining first name and last name into `full_name`.
- Creating surrogate keys such as `customer_key`, `film_key`, `store_key`, and `staff_key`.
- Creating date keys from rental, return, and payment dates.
- Mapping operational IDs to warehouse dimension keys.
- Calculating rental duration from `rental_date` and `return_date`.
- Detecting late returns by comparing actual rental duration with the expected film rental duration.
- Calculating payment amount and revenue.
- Handling missing return dates as films that have not been returned yet.
- Keeping `actor` and `film_actor` as optional future extensions for actor-level analysis.

### 5.3 Load

The transformed tables are loaded into the MySQL warehouse database:

`movie_rental_dw`

Dimension tables are loaded before fact tables because fact tables depend on dimension keys.

Loading order:

1. `dim_date`
2. `dim_customer`
3. `dim_film`
4. `dim_store`
5. `dim_staff`
6. `fact_rental`
7. `fact_payment`
8. `data_quality_report`

The notebook also includes a verification step after loading to confirm that all warehouse tables were created and that row counts are correct.

---

## 6. Data Quality Considerations

The ETL pipeline applies validation checks before loading the warehouse tables.

The validation checks include:

- Surrogate keys must be unique.
- Rental and payment IDs must not be duplicated.
- Required columns must not contain missing values.
- Fact table foreign keys must not be missing.
- Fact table foreign keys must exist in their related dimension tables.
- Payment amounts must not be negative.
- Rental duration must not be negative.
- Missing return dates are allowed because they represent rentals that have not been returned yet.

A `data_quality_report` table is created to store the validation results.

These checks help ensure that the warehouse is reliable for reporting and decision-making.

---

## 7. Conclusion

This project transforms a normalized movie rental OLTP database into a clean star-schema data warehouse.

The final warehouse contains two fact tables and five shared dimensions:

- `fact_rental`
- `fact_payment`
- `dim_date`
- `dim_customer`
- `dim_film`
- `dim_store`
- `dim_staff`

The ETL pipeline extracts the OLTP data, transforms it into analytical structures, validates data quality, loads the warehouse tables, and produces business insights through SQL queries and visualizations.

The warehouse supports analysis of rental activity, revenue, film popularity, customer behavior, store performance, staff performance, location activity, late returns, and time-based trends.

This design allows business managers to answer analytical questions without directly querying the operational OLTP database.
