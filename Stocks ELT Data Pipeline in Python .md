# PROJECT: Stocks ELT Data Pipeline in Python 
## Summary
1. Extract data from API Endpoint to temporary storage.
2. Perform data quality/validation checks with Python.
3. Load data to more permanent storage, where it is transformed to be ready for visualisation.
4. Visualise the data in dashboard format.
5. Automate this entire process.
## Motivations
I took data visualisation and storytelling courses during Bristol Data Week 2023, sparking an interest in understanding how companies handle data before creating final visualisations. 

The project aims to simulate a real company's process of data extraction, temporary storage, transformation, and visualisation, providing insights into industry data processing before end use.
## Approach
- Extract stocks’ open/close data from Polygon.io Stocks API to Amazon S3 bucket (Tech: **Python’s ’requests’ and ’boto3’ libraries**, flattening responses from JSON into a CSV file).
- Perform data quality/validation checks to reduce data burden (Tech: **Python’s ’pandas’** library).
- Execute **SQL** queries to load data into a column-based database in **Amazon Redshift Warehouse**.
- Perform data aggregation and calculated moving averages  in order to optimize a portfolio (Tech: **SQL** queries)
- Visualise results with interactive dashboards (Tech: **Tableau**)
- Automate the entire data pipeline, ensuring that the latest stocks data is fetched daily,
transformed, and stored in the database for further analysis and reporting (Tech: **Apache Airflow**)
## Project Description
### Data Source - Polygon.io REST API:
![image](https://github.com/nathann0209/pr0j3/assets/141504383/9bca0a17-6410-49cc-81d9-f1f78e15ce64)
- Provides REST endpoints that let you query the latest market data from all US stock exchanges.
- 5 API Calls / Minute.
- End of Day Data only (I used the free subscription).
- Responses in JSON object format.
### Architechture Diagram
![image](https://github.com/nathann0209/pr0j3/assets/141504383/ebe09438-e0cc-4dee-8a24-cc8954de639f)
1. Extract stocks data from polygon.io API endpoint, clean data using pandas.
2. Send to s3 bucket for temporary storage.
3. Load data from Amazon S3 to Amazon Redshift data warehouse.
4. Transform data in Amazon Redshift with SQL queries to a format that is fit for multiple data visualisations.
5. Perform data visuallisation to inform an optimal stock portfolio. 
## Technologies Used
- **Amazon Web Services (AWS)**:
    - Amazon S3 (Data Bucket)
    - Amazon Redshift (Data Warehouse)
- **Tableau Public:** Data Visualisation
- **Libraries:** boto3 (Querying API Endpoint), pandas (Data Cleaning)

## Stocks Table Structure
| Column | Description |
|---------|-------------|
| record_date | Stores the date of the requested stock's information |
| symbol | The stock symbol |
| stock_open | Open price of the stcok on the given date | 
| stock_high | Highest price of the stcok on the given date | 
| stock_low | Lowest price of the stcok on the given date | 
| stock_close | Close price of the stcok on the given date | 
| stock_volume | Volume of the stcok on the given date | 
| after_hours | After Hours price of the stcok on the given date | 
| pre_market | Pre Market price of the stcok on the given date | 

- The primary key to this table are the columns (record_date, symbol). 
## Important Code Snippets
### Requesting Data From API Endpoint
<pre>
def request_from_api(stock, dates_list): 
    extract_list = []
    for date in dates_list:
        # Request data from API in JSON format
        api_request = requests.get(f"https://api.polygon.io/v1/open-close/{stock}/{date}?adjusted=true&apiKey=XpmG_uxBbpfN988lFVLY7wo_lTOx3FiC")
        # Parse the JSON response received from the API into Python dictionary
        response_json = json.loads(api_request.content)
        # Convert extracted data values to a list and append the list to a list of lists
        extract_list.append(list(response_json.values()))
        # Pause program for 12 seconds, ensuring no more than 5 API requests per minute.
        time.sleep(12) 

    return extract_list
</pre>
### Writing Extracted Data to CSV:
<pre>
  def write_to_csv(rows):
    
    # Define the filename for the CSV file where the data will be exported.
    export_file = "export_proj3.csv"
    
    # Open the CSV file in write mode and write the contents of the rep_li list as a single row
    with open(export_file, 'a') as fp:
        csvw = csv.writer(fp)
        csvw.writerows(rows)
    fp.close()
    
    # Upload the CSV file (export_file) to an S3 bucket
    upload_to_s3(export_file)
</pre>
### Clean Data in Pandas
<pre>
  def data_quality_check(csv_file):
    # Read the CSV file into a DataFrame
    df = pd.read_csv(csv_file)

    # Remove rows with any null values
    df_cleaned = df.dropna()
    
    # Remove rows with 'NOT_FOUND' in the first column
    df_cleaned = df_cleaned[df.iloc[:, 0] != 'NOT_FOUND']
    
#     # Convert all other columns to float
#     df.iloc[:, 1:] = df.iloc[:, 1:].astype(float)

    # Remove duplicate rows
    df_cleaned = df_cleaned.drop_duplicates()
    
    # Write the cleaned data back to CSV file, overwriting original file
    df_cleaned.to_csv(csv_file, index=False)
    
</pre>
  
### Store data to s3 bucket
<pre>
  def upload_to_s3(exp_file):
    # load the aws_boto_credentials values
    parser = configparser.ConfigParser()
    parser.read("pipeline.conf")
    
    # retreive access key, secret key and bucket name from conf file
    access_key = parser.get("aws_boto_credentials",
    "access_key")
    secret_key = parser.get("aws_boto_credentials",
    "secret_key")
    bucket_name = parser.get("aws_boto_credentials",
    "bucket_name")
    
    # create a representation of the s3 bucket using boto3.client
    s3 = boto3.client('s3', aws_access_key_id=access_key, aws_secret_access_key=secret_key)
    
    # use boto3.client.upload_file to upload file into s3 bucket
    s3.upload_file(exp_file, bucket_name, exp_file)
</pre>
### Check if Data is Successfully Stored in Redshift Warehouse
<pre>
  def check_Redshift():    
    # Connection parameters
    host = 'pr0j3-cluster.cmmarghuvsrs.eu-west-2.redshift.amazonaws.com'
    port = '5439'
    dbname =  'dev'
    user = '#########'
    password = '#######'
    
    # Connect to Redshift
    conn = psycopg2.connect(host=host, port=port, dbname=dbname, user=user, password=password)

    # Create a cursor to execute SQL queries
    cursor = conn.cursor()

    # Execute a sample query
    cursor.execute('SELECT * FROM my_table')
    rows = cursor.fetchall()
    for row in rows:
        print(row)

    # Close cursor and connection
    cursor.close()
    conn.close()
</pre>
### Prepare Data For Visualisation
<pre>
Remove Unnecessary 'status' Column:

ALTER TABLE public.stock2_table
DROP status;

Calculate Moving Average:
  
SELECT symbol, record_date, stock_close, AVG(stock_close) OVER (ORDER BY record_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3
FROM public.stock2_table
ORDER BY symbol, record_date;
</pre>

## Exemplar Visualisations
- 2022 Candlestick Analysis (https://public.tableau.com/views/proj3book/Sheet2?:language=en-GB&publish=yes&:display_count=n&:origin=viz_share_link)
- 2022 Close Price Analysis (https://public.tableau.com/views/ClosePriceAnalysis/Sheet3?:language=en-GB&publish=yes&:display_count=n&:origin=viz_share_link)
- Moving Average Analysis (https://public.tableau.com/shared/MDHT2PM5G?:display_count=n&:origin=viz_share_link)
  
