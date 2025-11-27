# E-commerce Revenue Analysis Data Warehouse

This project implements a modern data engineering and analytics pipeline for E-commerce analytics, using the [Olist Brazil dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce/data?select=olist_order_items_dataset.csv). It demonstrates end-to-end transformation, modeling, and analysis workflows powered by dbt, PostgreSQL, and Python, enabling scalable, reproducible insights into E-commerce performance.

The project is focusing on tranforming from OLTP to OLAP based on KimBall Star-schema, so I did not complicate it by using Airflow or Python script for ETL, as the data source is not changing! So if you are looking for these tech stacks, please refer to my other projects!!!

## 🗂 Project Structure

```
.
├── Data/                    
│   ├── download_data.ipynb  # Notebook for downloading and extracting data
│   ├── create_raw_schema/   # SQL scripts for schema creation and data load
│   └── olist_*_dataset.csv  # CSV files (you can download this to skip the loading step)                  
├── ecommerce_dbt/           # dbt project: staging, marts, seeds, snapshots, macros
│   ├── dbt_project.yml
│   └── models/
│       ├── staging/         # Staging models
│       └── marts/           # Dimensional and fact models (main focus)
├── logs/                    
├── requirements.txt         # Python dependencies 
└── README.md                # Project documentation
```

## ⚙️ Technology Stack

- **Data Source**: Olist Brazil E-commerce Dataset
- **Programming Language**: Python 3.8+
- **Database**: PostgreSQL
- **Transformation Tool**: dbt (dbt-core, dbt-postgres)
- **Orchestration**: Notebooks / CLI

## 🧱 Data Architecture

### 1. Data Source

CSV files from the Olist Brazil dataset, including:
- Orders, customers, products, sellers, payments, reviews, geolocation, order items, and product categories.
- Raw data tables: ![Raw Data Table](image/raw_diagram.png)

### 2. Data Model

**Dimension Tables**  
- `dim_customer`: Customer identity and attributes  
- `dim_product`: Product information including names, category, seller,...
- `dim_seller`: Seller attributes: locations, seller_id,..
- `dim_date`: Calendar and business dates  
- `dim_location`: Geographical data

**Fact Tables**  
- `fct_orders`: Order-level transactions  
- `fct_reviews`: Customer reviews and ratings  
- `fct_payments`: Payment details

### 3. Data Flow

- **Source Layer**: CSV download/extraction from [Olist Brazil dataset-KaggleHub](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce/data?select=olist_order_items_dataset.csv)
- **Staging Layer**: Raw tables loaded via SQL scripts – refer to [`Data/create_raw_schema/create_raw_schema.sql`](./Data/create_raw_schema/create_raw_schema.sql) for the queries
- **Transformation Layer**: dbt models for staging and marts
- **Presentation Layer**: Cleaned fact/dim tables for BI and analytics

### 4. Data Quality Framework

- Null checks, foreign key validation
- dbt tests for schema integrity and business rules
- Logging and troubleshooting

## 🚀 Quick Start

### Prerequisites

- Python 3.8+  
- PostgreSQL (local or remote)  
- dbt-core and dbt-postgres  
- Kaggle credentials (for automated dataset download)

### Setup

1. **Environment Setup**  
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   pip install "dbt-core==1.*" dbt-postgres pandas kagglehub
   ```

2. **Download Data**  
   - Run `download_data.ipynb` to fetch and extract Olist dataset files  
   - Manual option: Place CSVs in `Data/` directory

3. **Load Data into Postgres**  
   ```bash
   psql -U postgres -c "CREATE DATABASE ecommerce_project;"
   psql -U postgres -d ecommerce_project -f Data/create_raw_schema/create_raw_schema.sql
   psql -U postgres -d ecommerce_project -f Data/create_raw_schema/load_into_raw.sql
   ```

4. **Configure dbt Profile**  
   Edit `~/.dbt/profiles.yml`:
   ```yaml
   your_profile_name:
     target: dev
     outputs:
       dev:
         type: postgres
         host: localhost
         user: postgres
         password: <your_password>
         port: 5432
         dbname: <your choice>
         schema: <your choice>
   ```

5. **Run dbt Models**  
   ```bash
   cd ecommerce_dbt
   dbt run --models staging/
   dbt run                # Runs all models
   dbt test               # Runs dbt tests
   ```

## 📊 Dashboard

Explore the interactive E-commerce Revenue Analysis dashboard on Tableau:  
[E-commerce Revenue Analysis Dashboard](https://public.tableau.com/views/E-commerceRevenueAnalysis_17585977814650/Dashboard1?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

![E-commerce Revenue Analysis Dashboard Screenshot](image/Dashboard_Project.png)
## 🧩 Project Components

- **Data Download Notebook**: Automates fetching and extraction of raw data
- **SQL Scripts**: Schema creation and data load for raw layer
- **dbt Project**: Modular models for data transformation, staging, and analytics marts
- **Logs**: dbt and process logs for auditing
- **Requirements File**: Pin Python dependencies (recommended for reproducibility)



## 🌐 Data Modeling Approach

### 1. Identify Business Processes

The E-commerce domain involves several key processes:
- **Ordering**: Customers place orders, tracked through status milestones.
- **Fulfillment**: Each order contains items (SKU, seller, price, freight).
- **Payment**: Orders may have multiple payments (type, installments, value).
- **Delivery**: Actual vs. estimated delivery dates.
- **Reviews** (optional): Ratings and comments post-delivery.

These processes are modeled as fact tables, supporting KPIs like revenue, AOV, delivery performance, and category/city analysis.

### 2. Define Fact Table Grain

To avoid many-to-many joins and ensure accurate analysis, facts are modeled at three grains:
- **Order-Item Grain**: One row per `(order_id, order_item_id)`  
    Table: `fact_order_items`
- **Order-Payment Grain**: One row per `(order_id, payment_sequential)`  
    Table: `fact_order_payments`
- **Order Grain**: One row per `order_id` (aggregated from items and payments)  
    Table: `fact_orders`

This multi-grain approach enables detailed analysis at item/payment level and clean roll-ups for dashboards.

### 3. Dimension Tables

Conformed dimensions provide descriptive context:
- **dim_customers**: Customer attributes (ID, unique ID, city, state)
- **dim_products**: Product details (ID, category, attributes)
- **dim_sellers**: Seller info (ID, location)
- **dim_dates**: Calendar attributes for time-based analysis
- **dim_location**: Geographical data (optional)

Surrogate keys are used for clean joins and future SCD support. Staging cleans data types and casing.

### 4. Fact Table Details

- **fact_order_items**: price, freight, product/seller/customer IDs, timing, delivery metrics
- **fact_order_payments**: payment type, installments, value, timing
- **fact_orders**: aggregated items/payments, delivery metrics, keys

## ⭐ Star Schema (Analytics Layer)
![KimBall_Star_Schema](image/Kimball_diagram.png)
### Fact Tables

| Fact Table            | Grain (One Row Per…)         | Primary Key                | Foreign Keys                                      | Core Measures (Examples)                                                      |
|-----------------------|------------------------------|----------------------------|---------------------------------------------------|-------------------------------------------------------------------------------|
| `fact_orders`         | `order_id` (order)           | `order_id`                 | `customer_id`, `order_date_key`                   | `items_count`, `total_price`, `total_freight`, `order_value`, `total_payment`, `payment_count`, `payment_methods_used`, `delivery_time_days`, `estimated_delivery_time_days` |
| `fact_order_items`    | `(order_id, order_item_id)` (item) | `(order_id, order_item_id)` | `order_id`, `customer_id`, `product_id`, `seller_id`, `order_date_key` | `price`, `freight_value`, `total_item_value`                                  |
| `fact_order_payments` | `(order_id, payment_sequential)` (payment) | `(order_id, payment_sequential)` | `order_id`, `customer_id`, `order_date_key`        | `payment_value`, `payment_installments` (with `payment_type` as descriptor)   |

> **Note:** `fact_orders` is built by aggregating items and payments first and then joining—this avoids item × payment duplication.

### Dimension Tables

| Dimension         | Surrogate PK     | Natural/Business Key      | Key Attributes (Samples)                                         | Notes                                              |
|-------------------|------------------|---------------------------|-------------------------------------------------------------------|----------------------------------------------------|
| `dim_customers`   | `customer_key`   | `customer_id`             | `customer_unique_id`, `customer_city`, `customer_state`           | Basic SCD columns included (`effective_date`, `end_date`, `is_current`) |
| `dim_products`    | `product_key`    | `product_id`              | `product_category`, `product_weight_g`, `product_length_cm`, `product_height_cm`, `product_width_cm` | Category translated to English                     |
| `dim_sellers`     | `seller_key`     | `seller_id`               | `seller_city`, `seller_state`, `seller_zip_code_prefix`           |                                                    |
| `dim_dates`       | `date_key`       | `date`                    | `year`, `quarter`, `month`, `day`, `day_name`, `month_name`, `is_weekend` | Used for `order_purchase_ts`                       |
| (optional) `dim_geolocation` | `geolocation_key` | `geolocation_zip_code_prefix` | `geolocation_city`, `geolocation_state`, `lat`, `lng`             | Only needed if you want zip-level mapping          |

### Relationship Matrix (Facts → Dimensions)

|                   | `dim_customers` | `dim_products` | `dim_sellers` | `dim_dates` | `dim_geolocation`* |
|-------------------|:---------------:|:--------------:|:-------------:|:-----------:|:------------------:|
| `fact_orders`     | ✅ via `customer_id` | –            | –            | ✅ via `order_date_key` | (via customer zip, if modeled) |
| `fact_order_items`| ✅              | ✅             | ✅            | ✅          | (via customer/seller zip, if modeled) |
| `fact_order_payments` | ✅          | –              | –            | ✅          | –                    |

---
Benefits of Transforming to OLAP database:
- Prevents item × payment duplication
- Enables analysis at appropriate detail
- Consistent dimensions across facts
- Viz-ready tables for BI tools




### 7. Data Lineage

```
CSV → raw → staging → marts → viz → Tableau
```
- **raw**: Original CSVs loaded into Postgres
- **staging**: Type/casing cleanup
- **marts**: Fact and dimension tables
- **viz**: Pre-joined tables for dashboards
- **Tableau**: KPIs, trends, maps

---

*Note: One unique identifier of a customer can be assigned to many customer_id.*


## 📬 Contact / License

Please reach out for questions and feedbacks. Any suggestions are appreciated!!!
