# Sql_scripting_for_projects 
## Airline Industry Studies for Google Data Analytics Capstone: Selecting, Filtering, Joining, Casting, When, Where, Aliasing, etc
## BigQuery for engine; data sources include "T100 Carrier by Domestic Segment" data from Bureau of Transportation Statistics and Aravind Ram Nathan's "List of US Airports"
## READ “T-100 Traffic Reporting Guide” before continuing

### Examine limited number of carriers for study base on sum of Departures Performed, count of Unique Carrier names select by segments served, or whatever metric is of interest
-- examine top 25 carriers by count of segment in original datafile
-- perform for each year of study to determine carriers for study then choose by metric performance over years of dat studied
SELECT 
    UNIQUE_CARRIER_NAME
FROM `dataanalyticscapstone-400804.Airline_Data_DupChk.Unclean_Carrier_2019` 
  GROUP BY UNIQUE_CARRIER_NAME
  ORDER BY 
   COUNT (UNIQUE_CARRIER_NAME) DESC
LIMIT 25

### Shrink data set to necessary data for analysis and begin cleaning
--create simplified table related to specific business question from original data and limited data set to subset 
--of table columns and carriers, change to lowercase, change troublesome names
SELECT
  MONTH AS num_month,
    YEAR AS num_year,
    ORIGIN AS origin,
    DEST AS dest,
    UNIQUE_CARRIER_NAME AS U_carrier_name,
    DEPARTURES_PERFORMED AS num_flights,
    RAMP_TO_RAMP AS ramp_to_ramp,
    AIR_TIME AS air_time
  FROM `dataanalyticscapstone-400804.Airline_Data_DupChk.Unclean_Carrier_2022`
  WHERE UNIQUE_CARRIER_NAME LIKE "Chosen1%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen2%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen3%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen4%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen5%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen6%"
###

###  Preserve needed columns but mask Airline names for posting studies since having to use an analog for tarmac delay with questionable accuracy.  Test with internal data if ever an option.
SELECT --create simplified table related to specific business question from original data and mask limited data set to study
    num_month,
    num_year,
    origin,
    dest,
    num_flights,
    ramp_to_ramp,
    air_time, 
      CASE 
      WHEN U_carrier_name = 'Chosen1' THEN 'Airline_A'
      WHEN U_carrier_name = 'Chosen3' THEN 'Airline_E'
      WHEN U_carrier_name = 'Chosen4' THEN 'Airline_C'
      WHEN U_carrier_name = 'Chosen5' THEN 'Airline_F'
      WHEN U_carrier_name = 'Chosen6' THEN 'Airline_D'
      ELSE 'Airline_2'
    END AS U_mask  
  FROM `dataanalyticscapstone-400804.Airline_Data_DupChk.Unclean_2022_ToMask`

### not finished 
