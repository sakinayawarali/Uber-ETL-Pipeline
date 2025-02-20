# Uber ETL Pipeline

This project was adapted by following the tutorial available here: [YouTube Video](https://www.youtube.com/watch?app=desktop&v=WpQECq5Hx9g)

## Parts of this Project
- Data Modeling
- Data Transformation

## Dataset Used
TLC Trip Record Data, which includes:
- Pick-up and drop-off dates/times
- Pick-up and drop-off locations
- Trip distances
- Itemized fares
- Rate types
- Payment types
- Driver-reported passenger counts

### Dataset Source
- [Uber ETL Pipeline Data](https://github.com/darshilparmar/uber-etl-pipeline-data-engineering-project/blob/main/data/uber_data.csv)
- More details about the dataset can be found:
  - [TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
  - [Data Dictionary](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf)

## Data Modeling
Data modeling was done using Lucidchart. The schema was inspired by the [GitHub repository](https://github.com/darshilparmar/uber-etl-pipeline-data-engineering-project/tree/main).

## Data Transformation
### Steps:
1. **Load the dataset**
    ```python
    import pandas as pd
    df = pd.read_csv("Data/uber_data.csv")
    ```
2. **Convert DateTime fields**
    ```python
    df['tpep_pickup_datetime'] = pd.to_datetime(df["tpep_pickup_datetime"])
    df['tpep_dropoff_datetime'] = pd.to_datetime(df["tpep_dropoff_datetime"])
    ```
3. **Remove duplicates and reset index**
    ```python
    df = df.drop_duplicates().reset_index(drop=True)
    df['trip_id'] = df.index
    ```
4. **Create Dimension Tables**
    - **Datetime Dimension**
      ```python
      datetime_dim = df[["tpep_pickup_datetime", "tpep_dropoff_datetime"]].reset_index(drop=True)
      datetime_dim["pick_hour"] = datetime_dim["tpep_pickup_datetime"].dt.hour
      datetime_dim["datetime_id"] = datetime_dim.index
      ```
    - **Passenger Count Dimension**
      ```python
      passenger_count_dim = df[["passenger_count"]].reset_index(drop=True)
      passenger_count_dim["passenger_count_id"] = passenger_count_dim.index
      ```
    - **Trip Distance Dimension**
      ```python
      trip_distance_dim = df[["trip_distance"]].reset_index(drop=True)
      trip_distance_dim["trip_distance_id"] = trip_distance_dim.index
      ```
    - **Rate Code Dimension**
      ```python
      rate_code_type = {
          1: "Standard Rate",
          2: "JFK",
          3: "Newark",
          4: "Nassau or Westchester",
          5: "Negotiated Fare",
          6: "Group Ride"
      }
      rate_code_dim = df[["RatecodeID"]].reset_index(drop=True)
      rate_code_dim["rate_code_id"] = rate_code_dim.index
      rate_code_dim["rate_code_name"] = rate_code_dim["RatecodeID"].map(rate_code_type)
      ```
    - **Location Dimensions (Pickup & Dropoff)**
      ```python
      pickup_location_dim = df[['pickup_longitude', 'pickup_latitude']].reset_index(drop=True)
      pickup_location_dim['pickup_location_id'] = pickup_location_dim.index
      dropoff_location_dim = df[['dropoff_longitude', 'dropoff_latitude']].reset_index(drop=True)
      dropoff_location_dim['dropoff_location_id'] = dropoff_location_dim.index
      ```
    - **Payment Type Dimension**
      ```python
      payment_type_name = {
          1:"Credit card",
          2:"Cash",
          3:"No charge",
          4:"Dispute",
          5:"Unknown",
          6:"Voided trip"
      }
      payment_type_dim = df[['payment_type']].reset_index(drop=True)
      payment_type_dim['payment_type_id'] = payment_type_dim.index
      payment_type_dim['payment_type_name'] = payment_type_dim['payment_type'].map(payment_type_name)
      ```
5. **Create Fact Table by Merging Dimensions**
    ```python
    fact_table = df.merge(passenger_count_dim, left_on='trip_id', right_on='passenger_count_id') \
                 .merge(trip_distance_dim, left_on='trip_id', right_on='trip_distance_id') \
                 .merge(rate_code_dim, left_on='trip_id', right_on='rate_code_id') \
                 .merge(pickup_location_dim, left_on='trip_id', right_on='pickup_location_id') \
                 .merge(dropoff_location_dim, left_on='trip_id', right_on='dropoff_location_id') \
                 .merge(datetime_dim, left_on='trip_id', right_on='datetime_id') \
                 .merge(payment_type_dim, left_on='trip_id', right_on='payment_type_id') \
                 [['trip_id','VendorID', 'datetime_id', 'passenger_count_id',
                   'trip_distance_id', 'rate_code_id', 'store_and_fwd_flag', 'pickup_location_id', 'dropoff_location_id',
                   'payment_type_id', 'fare_amount', 'extra', 'mta_tax', 'tip_amount', 'tolls_amount',
                   'improvement_surcharge', 'total_amount']]
    ```

## Conclusion
