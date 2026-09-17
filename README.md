# Netflix Movies and TV Shows ETL Pipeline

This project demonstrates a small data engineering pipeline built with PySpark and Spark SQL in Databricks. It reads a Netflix movies and TV shows CSV dataset from a Unity Catalog volume, standardizes the schema, masks payment-card data, and writes the transformed records to Azure Data Lake Storage Gen2 (ADLS Gen2).

## What was done

- Loaded 3,000 movie and TV show records from a CSV file with headers and inferred data types.
- Replaced spaces in column names with underscores so the fields are easier to query in Spark SQL.
- Registered the Spark DataFrame as a temporary SQL view named `movies`.
- Selected the fields needed for the processed dataset:
  - title
  - type
  - genre
  - release year
  - rating
  - duration
  - country
- Masked each card number by retaining only its first four digits and replacing the remaining visible portion with `****_****`.
- Validated that the transformation returned a Spark DataFrame and inspected its schema.
- Wrote the transformed data as headered CSV part files to an ADLS Gen2 container in overwrite mode.

## Pipeline

```text
CSV in a Databricks volume
          |
          v
    PySpark DataFrame
          |
          v
Column-name standardization
          |
          v
Spark SQL transformation and card masking
          |
          v
CSV files in ADLS Gen2
```

## Repository structure

```text
.
|-- data/
|   `-- Netflix_Movies_and_TV_Shows.csv  # Source dataset
|-- netflix-movies.ipynb                 # Databricks ETL notebook
`-- README.md
```

## Source data

The included CSV has the following columns:

| Column | Description |
| --- | --- |
| `Title` | Content title |
| `Type` | Movie or TV show |
| `Genre` | Content genre |
| `Release Year` | Year released |
| `Rating` | Content rating |
| `Duration` | Runtime or number of seasons |
| `Country` | Country associated with the title |
| `Card Number` | Sensitive field masked during transformation |

## Technologies used

- Python
- PySpark DataFrames
- Spark SQL
- Databricks and Unity Catalog Volumes
- Azure Data Lake Storage Gen2

## Configuration

The notebook currently uses these locations:

```python
# Input
/Volumes/10alytics_netflex_workspace/default/dataset/Netflix_Movies_and_TV_Shows.csv

# Output
abfss://esonetlixdata@esonetflixstorageaccount.dfs.core.windows.net/movies
```

Before running it in another environment, update the catalog, schema, volume, file name, and ADLS output URI in the notebook. The Databricks compute must have permission to read the Unity Catalog volume and write to the target storage container. ADLS credentials or a managed identity/service principal must already be configured on the workspace or cluster.

## How to run

1. Import `netflix-movies.ipynb` into a Databricks workspace.
2. Upload `data/Netflix_Movies_and_TV_Shows.csv` to the Unity Catalog volume referenced by `file_path`, or update the path to its actual location.
3. Configure access to the target ADLS Gen2 container.
4. Attach the notebook to a Spark-enabled compute resource.
5. Run the notebook cells in order.
6. Confirm that the final cell reports the output location and that the target `movies` directory contains CSV part files.

The write operation uses `mode("overwrite")`, so rerunning the final write replaces the existing output at that path.

## Output schema

| Output column | Transformation |
| --- | --- |
| `title` | Passed through from the source |
| `type` | Passed through from the source |
| `genre` | Passed through from the source |
| `release_year` | Renamed when spaces are replaced with underscores |
| `rating` | Passed through from the source |
| `duration` | Passed through from the source |
| `country` | Passed through from the source |
| `card_number_masked` | First four card-number digits followed by `****_****` |

> **Security note:** The raw CSV still contains complete card numbers. Treat it as sensitive data, restrict access to it, and avoid committing real payment-card data to source control. Masking is applied only to the transformed output.
