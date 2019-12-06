# Reddit Controversiality Analysis 

## Initial Attempts to read Data
### Uncompressed:
[json_to_sql](json_to_sql.ipynb)
### Compressed:
[Read_DF_SQLite](Read_DF_SQLite.ipynb)
## PySpark Jupyter Integration
demo: [spark](spark.ipynb)  

Part of this project utilizes PySpark and embedding of it into a Jupyter environment. 

### Requirements
Java 7+
Python 2.6+
Apache Spark 3.0
Hadoop 2.7 
Winutils.exe for Hadoop 2.6
Configure SPARK_HOME and HADOOP_HOME as system variables, and add them to Path 

### Run PySpark Instance
To help Jupyter Lab* bind to spark you can use findspark library.

### Option 2: Drive PySpark with Jupyter
To automatically open in Jupyter on startup add these system variables:
- set PYSPARK_DRIVER_PYTHON to jupyter
- set PYSPARK_DRIVER_PYTHON_OPTS to ‘lab’ 

### In-cell SQL
Insert the functions from [here](pys_sql.py)

## New Data: Big Query Extraction
[reddit-data-prep](DataPrep/reddit-data-prep.ipynb)

## EDA
[eda_vinay_controversial](eda_vinay_controversial.ipynb)
[csv_analysis](csv_analysis.ipynb)

## Controversiality Analysis
[one month data](Topic_model_world_news_one_month.ipynb)
[three month data](Topic_model_world_news_three_month.ipynb)

## Controversiality Prediction
[log_reg_controversy](log_reg_controversy.ipynb)

### Reddit Dataset 1
This dataset contains comments from Reddit.com, as collected by Reddit user Stuck_In_The_Matrix.
One torrent per year, going from 2005 to 2017 (the last one is of course incomplete).
Downloaded from https://files.pushshift.io/comments.
Example code for working with the dataset can be found @ https://github.com/dewarim/reddit-data-tools
Intened use is for scientific / non-commercial purposes.


### Reddit Dataset 2
This dataset contains is three months of big query data from r/worldnews 
