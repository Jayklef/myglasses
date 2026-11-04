# Eyeglasses Web Scraping ETL Pipeline - glass.com

A notebook-based ETL project that extracts eyeglasses product data from Glasses.com using Selenium and Beautiful Soup, transforms it with pandas, and loads it into PostgreSQL using SQLAlchemy.

## Project Overview

The pipeline follows three stages:

1. **Extract:** Open product listing pages with Selenium and parse the rendered HTML using Beautiful Soup.
2. **Transform:** Organize the extracted data into a pandas DataFrame and convert product prices to numeric values.
3. **Load:** Write the cleaned records to a PostgreSQL table.

## Technology Stack

- Python
- Jupyter Notebook
- Selenium
- Beautiful Soup
- pandas
- SQLAlchemy
- Psycopg 3
- PostgreSQL
- Google Chrome

## Extracted Fields

| Column | Description |
|---|---|
| `product_badge` | Product labels, combined into a comma-separated string |
| `product_colors` | Displayed number of available colors |
| `product_code` | Product name or model label shown on the listing |
| `product_brand` | Product brand |
| `product_price` | Listed price, converted to a numeric value |

Missing product fields are recorded as `None`.

## Prerequisites

- Python 3.10 or later
- Google Chrome
- A running PostgreSQL server
- An existing PostgreSQL database
- Internet access for the website and initial driver setup

Selenium normally manages ChromeDriver automatically.

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on macOS or Linux:

```bash
source .venv/bin/activate
```

Activate it on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install the dependencies:

```bash
python -m pip install pandas selenium beautifulsoup4 requests sqlalchemy "psycopg[binary]" notebook
```

## Database Setup

Create the database in PostgreSQL using pgAdmin or a SQL client:

```sql
CREATE DATABASE glasses;
```

Configure the connection in the notebook:

```python
from sqlalchemy import create_engine, URL

db_connection = URL.create(
    drivername="postgresql+psycopg",
    username="YOUR_USERNAME",
    password="YOUR_PASSWORD",
    host="localhost",
    port=5432,
    database="glasses",
)

engine = create_engine(db_connection)
```

Replace the placeholder credentials with your local settings. Remove real credentials and connection outputs before committing the notebook.

## Running the Pipeline

Start Jupyter Notebook:

```bash
jupyter notebook
```

Open the project notebook and run its cells in order:

1. Import the libraries.
2. Configure and start Chrome.
3. Extract product details from the listing pages.
4. Create the raw DataFrame.
5. Copy and clean the data.
6. Configure the PostgreSQL connection.
7. Load the cleaned DataFrame.
8. Query the database to inspect the results.

The extraction currently attempts ten listing pages:

```python
total_pages = 10
```

It constructs URLs using:

```text
https://www.glasses.com/gl-us/eyeglasses?pageNumber=1
```

Verify that changing `pageNumber` returns different products before treating the result as a complete multi-page extraction.

## Data Transformation

The pipeline preserves the raw DataFrame and creates a working copy:

```python
clean_glasses_df = glass_df.copy()
```

Prices are cleaned by removing the currency symbol and surrounding whitespace before conversion to `float`.

The PostgreSQL price column is created as `NUMERIC(12, 2)`.

## Database Loading

The cleaned data is loaded into:

```text
public.glasses
```

The load runs inside a SQLAlchemy transaction:

```python
with engine.begin() as connection:
    clean_glasses_df.to_sql(
        name="glasses",
        con=connection,
        schema="public",
        if_exists="replace",
        index=False,
        dtype={"product_price": Numeric(12, 2)},
    )
```

**Each run replaces the destination table.** Existing rows and table constraints are removed when the table is dropped and recreated.

## Verifying the Load

Retrieve the stored data:

```sql
SELECT * FROM public.glasses;
```

Check the number of loaded rows:

```sql
SELECT COUNT(*) AS total_products
FROM public.glasses;
```

Compare this count with:

```python
len(clean_glasses_df)
```

Matching counts confirm the number of stored records, but do not establish that the source extraction is complete or free of duplicates.

## Current Limitations

- Pagination behavior has not been independently verified.
- The scraper uses a fixed delay, which may be insufficient on slower connections.
- Only products present in the rendered HTML at extraction time are collected.
- Changes to the website's HTML classes may break extraction.
- Product URLs are not currently retained, limiting reliable duplicate detection.
- The pipeline does not yet implement retries, upserts, or automated scheduling.
- Replacing the database table does not preserve historical product data.

## Planned Improvements

- Replace fixed delays with explicit Selenium waits.
- Verify pagination and detect the end of the listing dynamically.
- Retain product URLs and extraction timestamps.
- Add validation for empty results, missing fields, and duplicate products.
- Guarantee browser cleanup using `try/finally`.
- Move database credentials into environment variables.
- Add structured logging and retry handling.
- Implement incremental loading or upserts.
- Separate the notebook workflow into reusable pipeline modules.

## Responsible Use

Review the website's terms and scraping guidance before running the pipeline. Keep request frequency reasonable and stop if access is denied.