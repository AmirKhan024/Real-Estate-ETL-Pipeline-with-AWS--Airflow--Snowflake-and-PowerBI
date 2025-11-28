# Redfin Real Estate Analytics Pipeline

![NWZrBEnJ6Us-HD](https://github.com/user-attachments/assets/26892bd9-23fd-43b4-addd-066f80cd7b69)


## 📊 Project Overview

An end-to-end data engineering and analytics solution that extracts real estate market data from Redfin, processes it through AWS services, loads it into Snowflake, and visualizes insights in Power BI. This pipeline demonstrates modern cloud data architecture with automated ETL processes orchestrated by Apache Airflow.

## 🏗️ Architecture

The project implements a complete data pipeline with the following components:

1. **Data Source**: Redfin public real estate market data
2. **Extraction**: Python scripts extract data from Redfin's public datasets
3. **Storage Layer**: AWS S3 buckets for raw and transformed data
4. **Transformation**: Python-based data cleaning and transformation
5. **Orchestration**: Apache Airflow running on AWS EC2 for workflow management
6. **Event Notification**: Amazon SQS for S3 event notifications
7. **Data Warehouse**: Snowflake for analytics and data storage
8. **Data Ingestion**: Snowpipe for automated data loading triggered by SQS
9. **Visualization**: Power BI dashboards for business insights

## 🛠️ Technologies Used

- **Cloud Platform**: AWS (S3, EC2, SQS)
- **Workflow Orchestration**: Apache Airflow
- **Programming Language**: Python 3.10
- **Data Warehouse**: Snowflake
- **Data Ingestion**: Snowpipe with SQS event notifications
- **Visualization**: Power BI
- **Libraries**: pandas, boto3, awscli

## 🚀 Features

### Data Pipeline
- **Automated Data Extraction**: Scheduled extraction from Redfin's public data repository
- **Data Transformation**: 
  - Removal of special characters (commas in city names)
  - Date parsing and temporal feature engineering (year/month extraction)
  - Data cleaning and null value handling
  - Column selection and filtering
- **Cloud Storage**: Dual-bucket architecture for raw and transformed data
- **Error Handling**: Retry logic and failure notifications

### Data Warehouse
- Structured database schema in Snowflake
- Automated data ingestion via Snowpipe
- External staging with S3 integration
- Optimized table structure for analytics

### Analytics Dashboard
Interactive Power BI dashboard featuring:
- **Average Median Sale Price by City**: Bar chart visualization
- **Average Median Sale Price Trends**: Time series analysis by city and year
- **Property Type Distribution**: Donut chart showing market composition
- **Average Median DOM by City**: Days on market analysis
- **Inventory Trends**: Stacked area chart showing inventory over time
- **Geographic Distribution**: Interactive map with median price per square foot
- **City Filter**: Dynamic filtering across all visualizations

## 📋 Prerequisites

- AWS Account with S3 and EC2 access
- Snowflake account
- Python 3.8+
- Apache Airflow 2.x
- Power BI Desktop

## ⚙️ Setup Instructions

### 1. AWS Configuration

**Create S3 Buckets:**
```bash
# Raw data bucket
aws s3 mb s3://store-raw-data-yml

# Transformed data bucket
aws s3 mb s3://redfin-transform-zone-yml
```

**Set up EC2 Instance:**
- Launch an Ubuntu EC2 instance (t2.medium or higher recommended)
- Configure security groups to allow HTTP/HTTPS access
- Install Apache Airflow and dependencies

### 2. Airflow Setup on EC2

**Install Dependencies and Airflow:**
```bash
# Update system packages
sudo apt update

# Install Python pip
sudo apt install python3-pip

# Install Python virtual environment
sudo apt install python3.12-venv

# Create virtual environment
python3 -m venv redfin_venv

# Activate virtual environment
source redfin_venv/bin/activate

# Install required Python packages
pip install pandas
pip install boto3
pip install --upgrade awscli
pip install apache-airflow

# Verify installations
airflow version
python3 --version

# Configure AWS credentials
aws configure
# Enter your AWS Access Key ID, Secret Access Key, region, and output format

# Start Airflow in standalone mode
airflow standalone
```

<img width="1227" height="941" alt="image" src="https://github.com/user-attachments/assets/a1d9e778-5ff8-4f32-875f-5f22cbe15305" />


**Deploy DAG:**
1. Copy `redfin_analytics.py` to `~/airflow/dags/`
2. Update AWS credentials in the script or use IAM roles (recommended)
3. Configure S3 bucket names if different
4. Airflow will automatically detect and load the DAG

### 3. Snowflake Configuration

1. Create a Snowflake account
2. Run the SQL script from `snowflake/setup_script.sql`
3. Update AWS credentials in the external stage configuration
4. Configure Snowpipe for automated ingestion

```sql
CREATE OR REPLACE STAGE redfin_database_1.external_stage_schema.redfin_ext_stage_yml 
    url="s3://redfin-transform-zone-yml/"
    credentials=(aws_key_id='YOUR_AWS_KEY_ID'
    aws_secret_key='YOUR_AWS_SECRET_KEY')
```

**Get Snowpipe Notification Channel:**
After creating the Snowpipe, run this command in Snowflake to get the SQS ARN:
```sql
DESC PIPE redfin_database_1.snowpipe_schema.redfin_snowpipe;
```
Copy the `notification_channel` value - this is the SQS ARN you'll need for S3 event notifications.

### 3.1 Configure S3 Event Notifications with Amazon SQS

1. Go to AWS S3 Console
2. Select the `redfin-transform-zone-yml` bucket
3. Navigate to **Properties** tab
4. Scroll to **Event notifications** section
5. Click **Create event notification**
6. Configure the event:
   - **Event name**: SnowpipeTrigger
   - **Event types**: Check "All object create events" (s3:ObjectCreated:*)
   - **Destination**: Select "SQS queue"
   - **SQS queue**: Enter the ARN from Snowpipe's notification_channel
7. Click **Save changes**

This connects your S3 bucket to Snowpipe - whenever a new file is uploaded to the transformed data bucket, Snowflake will automatically ingest it.

### 4. Power BI Setup

1. Open Power BI Desktop
2. Connect to Snowflake data warehouse:
   - Get Data → Database → Snowflake
   - Enter your Snowflake account URL
   - Authenticate with your credentials
3. Load the `redfin_table` from the database
4. Create visualizations as shown in the dashboard preview

   <img width="1238" height="690" alt="image" src="https://github.com/user-attachments/assets/ed806b71-7364-4c04-8b0b-448fe0fc6d2c" />


## 🔄 Pipeline Workflow

1. **Extract**: Airflow DAG triggers the extraction task
   - Downloads compressed TSV file from Redfin's S3 bucket
   - Converts to CSV format with timestamp

2. **Transform**: Python transformation task processes the data
   - Cleans and validates data
   - Creates temporal features
   - Removes null values

3. **Load**: 
   - Raw data uploaded to `store-raw-data-yml` bucket
   - Transformed data uploaded to `redfin-transform-zone-yml` bucket

4. **Ingest**: Automated ingestion via Amazon SQS and Snowpipe
   - S3 event notification triggers Amazon SQS when new file arrives
   - SQS notifies Snowpipe about the new file
   - Snowpipe automatically loads data into Snowflake table
   - Maintains data integrity and schema validation

5. **Visualize**: Power BI refreshes to show latest data
   - Real-time insights on real estate trends
   - Interactive filtering and drill-down capabilities

## 📊 Data Schema

The Snowflake table includes the following key fields:

**Temporal Fields:**
- `period_begin` (DATE) - Start date of the reporting period
- `period_end` (DATE) - End date of the reporting period
- `period_duration` (INT) - Duration of the period
- `period_begin_in_years` (STRING) - Year of period start
- `period_end_in_years` (STRING) - Year of period end
- `period_begin_in_months` (STRING) - Month of period start (Jan, Feb, etc.)
- `period_end_in_months` (STRING) - Month of period end

**Geographic Fields:**
- `city` (STRING) - City name
- `state` (STRING) - State name
- `state_code` (STRING) - State abbreviation
- `region_type` (STRING) - Type of region
- `region_type_id` (INT) - Region type identifier

**Property Information:**
- `property_type` (STRING) - Type of property (Single Family, Condo, Townhouse, etc.)
- `property_type_id` (INT) - Property type identifier

**Market Metrics:**
- `median_sale_price` (FLOAT) - Median sale price in USD
- `median_list_price` (FLOAT) - Median listing price in USD
- `median_ppsf` (FLOAT) - Median price per square foot
- `median_list_ppsf` (FLOAT) - Median list price per square foot
- `homes_sold` (FLOAT) - Number of homes sold
- `inventory` (FLOAT) - Available inventory
- `months_of_supply` (FLOAT) - Months of supply available
- `median_dom` (FLOAT) - Median days on market
- `avg_sale_to_list` (FLOAT) - Average sale to list price ratio
- `sold_above_list` (FLOAT) - Percentage sold above list price

**Metadata:**
- `table_id` (INT) - Table identifier
- `is_seasonally_adjusted` (STRING) - Whether data is seasonally adjusted
- `parent_metro_region_metro_code` (STRING) - Parent metro region code
- `last_updated` (DATETIME) - Last update timestamp


## 🎯 Use Cases

This pipeline and dashboard can answer business questions such as:
- Which cities have the highest median sale prices?
- How have real estate prices trended over time?
- What is the distribution of property types in the market?
- Which areas have the shortest/longest days on market?
- How does inventory vary seasonally across different cities?
- What is the price per square foot across different regions?

## 🔧 Customization

### Modify Data Source
Update the URL in `redfin_analytics.py`:
```python
# Change to different Redfin dataset
url_by_county = 'https://redfin-public-data.s3.us-west-2.amazonaws.com/redfin_market_tracker/county_market_tracker.tsv000.gz'
```

### Adjust Schedule
Modify the DAG schedule in `redfin_analytics.py`:
```python
with DAG('redfin_analytics_dag',
        default_args=default_args,
        schedule_interval='@weekly',  # or '@daily', '@monthly', etc.
        catchup=False) as dag:
```

### Add Transformations
Extend the `transform_data` function with additional data processing logic.

## 📈 Future Enhancements

- [ ] Add data quality checks and validation rules
- [ ] Implement incremental loads instead of full refreshes
- [ ] Add email notifications for pipeline failures
- [ ] Create additional visualizations for predictive analytics
- [ ] Integrate machine learning models for price predictions
- [ ] Add dbt for transformation layer
- [ ] Implement data lineage tracking
- [ ] Add unit tests for transformation logic
